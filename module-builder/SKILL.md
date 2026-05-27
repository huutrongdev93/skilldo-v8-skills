---
name: module-builder
description: Hướng dẫn tạo Module admin CRUD hoàn chỉnh cho Plugin/Theme trong SkillDo CMS v8 - Model, Route, Form, Table, Controller, View, Menu và Bootstrap hooks.
---

# Tạo Module Admin CRUD - SkillDo CMS v8

Hướng dẫn tạo module quản trị hoàn chỉnh (CRUD) gồm: Model, Route, Form, Table, Controller, View, Menu.

**Ví dụ:** Module **Quản lý Dự án (Projects)** trong plugin `my-plugin`.

## Cấu Trúc Thư Mục

```
plugins/my-plugin/
├── app/
│   ├── Controllers/Admin/
│   │   └── ProjectController.php
│   ├── Models/
│   │   └── Project.php
│   └── Modules/Admin/Project/
│       ├── Form.php
│       ├── Table.php
│       └── Logger.php          # (tùy chọn)
├── bootstrap/
│   └── projects.php            # Đăng ký hooks
├── routes/
│   └── admin.php
└── views/admin/projects/
    ├── index.blade.php
    └── save.blade.php
```

---

## Bước 1. Tạo Model

*Đường dẫn: `app/Models/Project.php`*

```php
<?php
namespace MyPlugin\Models;

class Project extends \SkillDo\Cms\Models\Model
{
    protected string $table = 'projects';

    protected string $primaryKey = 'id';
}
```

---

## Bước 2. Tạo Route Admin

*Đường dẫn: `routes/admin.php`*

```php
<?php
use SkillDo\Support\Facades\Route;

Route::middleware('auth:admin')->prefix('admin/projects')->group(function() {
    $controller = \MyPlugin\Controllers\Admin\ProjectController::class;

    Route::match(['get','post'], '/', $controller.'@index')->name('admin.projects.index');
    Route::match(['get','post'], '/add', $controller.'@add')->name('admin.projects.add');
    Route::match(['get','post'], '/edit/{id}', $controller.'@edit')->where('id', '[0-9]+')->name('admin.projects.edit');
});
```

> CMS v8 tự động quét file `routes/admin.php` trong thư mục `routes` của plugin.

---

## Bước 3. Tạo Form

*Đường dẫn: `app/Modules/Admin/Project/Form.php`*

```php
<?php
namespace MyPlugin\Modules\Admin\Project;

use MyPlugin\Models\Project;
use SkillDo\Cms\FormAdmin\FormAdmin;
use SkillDo\Cms\Support\Admin;
use SkillDo\Cms\Support\Language;
use SkillDo\Cms\Support\Url;
use SkillDo\Validate\Rule;

class Form
{
    /**
     * Định nghĩa các field trong form add/edit
     */
    static function fields(FormAdmin $form): FormAdmin
    {
        $form->setModel(Project::class);

        // Cột trái - các field hỗ trợ đa ngôn ngữ
        $form->lang()
            ->addGroup('info', 'Thông Tin Dự Án')
            ->text('title', [
                'label'       => 'Tên Dự Án',
                'validations' => [
                    Language::default() => Rule::make()->notEmpty()
                ]
            ])
            ->wysiwygShort('excerpt', ['label' => 'Mô tả ngắn'])
            ->wysiwyg('content', ['label' => 'Nội dung chi tiết']);

        // Cột phải - media
        $form->right()
            ->addGroup('media', 'Ảnh đại diện')
            ->image('image', ['label' => 'Upload Hình']);

        // Cột phải - SEO (tùy chọn)
        $form->right()
            ->addGroup('seo', 'SEO')
            ->text('slug', ['label' => 'Đường dẫn'])
            ->text('seo_title', ['label' => 'SEO Title'])
            ->text('seo_keywords', ['label' => 'SEO Keywords'])
            ->textarea('seo_description', ['label' => 'SEO Description']);

        return $form;
    }

    /**
     * Định nghĩa các button action (Lưu, Thêm mới, Quay lại)
     */
    static function buttons(FormAdmin $form): FormAdmin
    {
        $buttons = [];

        if(Admin::isPage('projects_add'))
        {
            $buttons[] = Admin::button('save');
            $buttons[] = Admin::button('back', ['href' => Url::admin('projects')]);
        }

        if(Admin::isPage('projects_edit'))
        {
            $buttons[] = Admin::button('save');
            $buttons[] = Admin::button('add', [
                'href'    => Url::admin('projects/add'),
                'text'    => '',
                'tooltip' => trans('button.add')
            ]);
            $buttons[] = Admin::button('back', [
                'href'    => Url::admin('projects'),
                'text'    => '',
                'tooltip' => trans('button.back')
            ]);
        }

        $form->setButtons($buttons);

        return $form;
    }
}
```

### Cấu trúc FormAdmin

| Method | Mô tả |
|--------|--------|
| `$form->setModel(Model::class)` | **Bắt buộc** — Gán model để CMS tự xử lý CRUD |
| `$form->lang()` | Trả về builder cho cột trái (hỗ trợ đa ngôn ngữ) |
| `$form->left()` | Trả về builder cho cột trái (không đa ngôn ngữ) |
| `$form->right()` | Trả về builder cho cột phải |
| `->addGroup($id, $label)` | Tạo group (collapsible box) chứa các field |
| `Admin::isPage('module_add')` | Kiểm tra đang ở trang thêm mới |
| `Admin::isPage('module_edit')` | Kiểm tra đang ở trang chỉnh sửa |

### Validation cho field đa ngôn ngữ

Khi field hỗ trợ đa ngôn ngữ (nằm trong `$form->lang()`), validation cần chỉ định theo ngôn ngữ:

```php
'validations' => [
    Language::default() => Rule::make()->notEmpty()
]
```

---

## Bước 4. Tạo Table

*Đường dẫn: `app/Modules/Admin/Project/Table.php`*

```php
<?php
namespace MyPlugin\Modules\Admin\Project;

use MyPlugin\Models\Project;
use SkillDo\Cms\Form\Form;
use SkillDo\Cms\Support\Admin;
use SkillDo\Cms\Support\Url;
use SkillDo\Cms\Table\Columns\ColumnEdit;
use SkillDo\Cms\Table\Columns\ColumnImage;
use SkillDo\Cms\Table\Columns\ColumnText;
use SkillDo\Cms\Table\Columns\ColumnBadge;
use SkillDo\Cms\Table\SKDObjectTable;
use SkillDo\Database\Eloquent\Builder;
use SkillDo\Http\Request;

class Table extends SKDObjectTable
{
    protected string $module = 'projects';

    protected mixed $model = Project::class;

    /**
     * Định nghĩa các cột hiển thị
     */
    function getColumns()
    {
        $this->_column_headers = [];

        // Checkbox chọn hàng loạt
        $this->_column_headers['cb'] = 'cb';

        // Cột ảnh
        $this->_column_headers['image'] = [
            'label'  => 'Ảnh',
            'column' => fn($item, $args) => ColumnImage::make('image', $item, $args)->size(50)
        ];

        // Cột tiêu đề (có link edit)
        $this->_column_headers['title'] = [
            'label'  => 'Tên Dự Án',
            'column' => fn($item, $args) => ColumnText::make('title', $item, $args)->title()
        ];

        // Cột thứ tự (editable inline)
        $this->_column_headers['order'] = [
            'label'  => trans('table.order'),
            'column' => fn($item, $args) => ColumnEdit::make('order', $item, $args)
        ];

        // Cột ngày tạo
        $this->_column_headers['created'] = [
            'label'  => 'Ngày tạo',
            'column' => fn($item, $args) => ColumnText::make('created', $item, $args)->datetime('d/m/Y')
        ];

        // Cột action (edit/delete buttons)
        $this->_column_headers['action'] = trans('table.action');

        return apply_filters("manage_projects_columns", $this->_column_headers);
    }

    /**
     * Các button action cho mỗi hàng
     */
    function actionButton($item, $module, $table): array
    {
        $listButton = [];

        // Nút sửa
        $listButton[] = Admin::button('blue', [
            'href' => Url::admin('projects/edit/'.$item->id),
            'icon' => Admin::icon('edit')
        ]);

        // Nút xóa
        $listButton[] = Admin::btnDelete([
            'id'          => $item->id,
            'model'       => $this->model,
            'module'      => $this->module,
            'description' => trans('admin::message.page.confirmDelete', [
                'title' => html_escape($item->title)
            ])
        ]);

        return apply_filters('admin_projects_table_columns_action', $listButton);
    }

    /**
     * Button ở header table (Thêm mới, Reload)
     */
    function headerButton(): array
    {
        $buttons = [];
        $buttons[] = Admin::button('add', ['href' => Url::admin('projects/add')]);
        $buttons[] = Admin::button('reload');
        return $buttons;
    }

    /**
     * Form tìm kiếm trong header table
     */
    function headerSearch(Form $form, Request $request): Form
    {
        $form->text('keyword', [
            'placeholder' => trans('table.search.keyword').'...'
        ], $request->input('keyword'));

        return $form;
    }

    /**
     * Xử lý query filter khi tìm kiếm
     */
    public function queryFilter(Builder $query, Request $request): Builder
    {
        $keyword = $request->input('keyword');

        if (!empty($keyword))
        {
            $query->where('title', 'like', '%'.$keyword.'%');
        }

        return $query;
    }

    /**
     * Sắp xếp mặc định cho danh sách
     */
    public function queryDisplay(Builder $query, Request $request, $data = []): Builder
    {
        $query = parent::queryDisplay($query, $request, $data);

        $query->orderBy('order')->orderBy('created', 'desc');

        return $query;
    }
}
```

### Các loại Column có sẵn

| Column Class | Mô tả | Methods |
|-------------|--------|---------|
| `ColumnText` | Hiển thị text | `->title()`, `->datetime($format)`, `->number()`, `->color($color)`, `->description($cb)` |
| `ColumnImage` | Hiển thị ảnh | `->size($px)`, `->width($px)`, `->height($px)`, `->circular()`, `->link($url)` |
| `ColumnEdit` | Text editable inline | `->class($class)`, `->isEdit($cb)` |
| `ColumnBadge` | Badge trạng thái | `->color($cb)`, `->label($cb)`, `->class($cb)`, `->attributes($cb)` |
| `ColumnView` | Custom HTML | `->html($callback)` |

### Các method Table quan trọng

| Method | Signature | Mô tả |
|--------|-----------|--------|
| `getColumns()` | `function getColumns()` | Định nghĩa các cột bảng |
| `actionButton()` | `function actionButton($item, $module, $table): array` | Button action mỗi hàng |
| `headerButton()` | `function headerButton(): array` | Button ở header |
| `headerSearch()` | `function headerSearch(Form $form, Request $request): Form` | Form tìm kiếm |
| `headerFilter()` | `function headerFilter(Form $form, Request $request)` | Form filter |
| `queryFilter()` | `public function queryFilter(Builder $query, Request $request): Builder` | Query khi search/filter |
| `queryDisplay()` | `public function queryDisplay(Builder $query, Request $request, $data = []): Builder` | Sắp xếp hiển thị |
| `bulkAction()` | `function bulkAction(): array` | Hành động hàng loạt |

---

## Bước 5. Tạo Controller

*Đường dẫn: `app/Controllers/Admin/ProjectController.php`*

```php
<?php
namespace MyPlugin\Controllers\Admin;

use Admin\Supports\FormAdminHelper;
use MyPlugin\Models\Project;
use MyPlugin\Modules\Admin\Project\Table;
use SkillDo\Cms\Controller;
use SkillDo\Cms\Support\Admin;
use SkillDo\Cms\Support\Cms;
use SkillDo\Http\Request;

class ProjectController extends Controller
{
    function __construct()
    {
        parent::__construct();
        Cms::setData('module', 'projects');
    }

    public function index(Request $request)
    {
        Cms::setData('table', new Table());
        return Cms::view('my-plugin::admin/projects/index');
    }

    public function add(Request $request)
    {
        Cms::setData('form', FormAdminHelper::getForm('projects'));
        return Cms::view('my-plugin::admin/projects/save');
    }

    public function edit(Request $request, $id = '')
    {
        $object = Project::find($id);

        if(noItems($object))
        {
            return Admin::pageNotFound();
        }

        Cms::setData('object', $object);
        Cms::setData('form', FormAdminHelper::getForm('projects', $object));

        return Cms::view('my-plugin::admin/projects/save');
    }
}
```

### Quy tắc Controller

- `Cms::setData('module', 'projects')` — **bắt buộc** trong constructor, trùng với `$module` trong Table
- `Cms::setData('table', new Table())` — truyền Table instance cho trang index
- `FormAdminHelper::getForm('projects')` — tạo form mới (add)
- `FormAdminHelper::getForm('projects', $object)` — tạo form có data (edit)
- Tên hook form: `manage_{module}_input` — `{module}` = giá trị `Cms::setData('module', ...)`
- View namespace: `{plugin-folder}::path/to/view`

---

## Bước 6. Tạo Views

### 6.1 Trang danh sách (index)

*Đường dẫn: `views/admin/projects/index.blade.php`*

```blade
{!! Admin::partial('resources/page-default/page-index', [
    'name'   => 'Quản lý Dự Án',
    'module' => $module,
    'model'  => \MyPlugin\Models\Project::class,
    'table'  => $table,
]) !!}
```

### 6.2 Trang thêm/sửa (save)

*Đường dẫn: `views/admin/projects/save.blade.php`*

```blade
{!! Admin::partial('resources/page-default/page-save', [
    'module' => $module,
    'model'  => \MyPlugin\Models\Project::class,
    'object' => (isset($object)) ? $object : []
]) !!}
```

### Tham số View

**page-index:**

| Param | Type | Mô tả |
|-------|------|--------|
| `name` | string | Tiêu đề trang |
| `module` | string | Tên module |
| `model` | string | Class model |
| `table` | SKDObjectTable | Instance table |

**page-save:**

| Param | Type | Mô tả |
|-------|------|--------|
| `module` | string | Tên module |
| `model` | string | Class model |
| `object` | object/array | Dữ liệu object khi edit, `[]` khi add |

---

## Bước 7. Đăng Ký Menu Admin

Tạo class chứa logic đăng ký menu:

*Đường dẫn: `app/Services/AdminService.php`*

```php
<?php
namespace MyPlugin\Services;

use SkillDo\Cms\Menu\AdminMenu;

class AdminService
{
    static public function navigation(): void
    {
        AdminMenu::add('projects', 'Quản lý Dự Án', 'projects', [
            'icon'     => '<i class="fa-solid fa-briefcase"></i>',
            'position' => 30
        ]);

        // Submenu (tùy chọn)
        AdminMenu::addSub('projects', 'projects', 'Danh sách', 'projects');
        AdminMenu::addSub('projects', 'projects-add', 'Thêm mới', 'projects/add');
    }
}
```

Đăng ký hook trong bootstrap:

*Đường dẫn: `bootstrap/projects.php`*

```php
<?php
use MyPlugin\Services\AdminService;

add_action('admin_navigation', [AdminService::class, 'navigation']);
```

### AdminMenu API

```php
// Thêm menu chính
AdminMenu::add($key, $title, $slug, [
    'icon'     => '<i class="..."></i>',   // Icon FontAwesome
    'position' => 30,                       // Vị trí (số nhỏ = trên)
    'hidden'   => false,                    // Ẩn menu
    'count'    => 0,                        // Badge count
]);

// Thêm submenu
AdminMenu::addSub($parentKey, $key, $title, $slug, [
    'position' => 1,
    'count'    => 0,
]);

// Xóa menu
AdminMenu::remove($key);

// Kiểm tra menu tồn tại
AdminMenu::has($key);
```

**Position mặc định:**

| Key | Position |
|-----|----------|
| `home` | 2 |
| `page` | 11 |
| `post` | 21 |
| `galleries` | 31 |
| `theme` | 41 |
| `plugins` | 51 |
| `system` | 61 |
| `user` | 71 |

---

## Bước 8. Đăng Ký Hooks (Bootstrap)

*Đường dẫn: `bootstrap/projects.php`*

```php
<?php
use MyPlugin\Modules\Admin\Project\Form;
use MyPlugin\Modules\Admin\Project\Logger;

/*
|--------------------------------------------------------------------------
| Hook Form
|--------------------------------------------------------------------------
*/
add_filter('manage_projects_input', [Form::class, 'fields']);
add_filter('manage_projects_input', [Form::class, 'buttons']);

/*
|--------------------------------------------------------------------------
| Hook Logger (tùy chọn)
|--------------------------------------------------------------------------
*/
add_action('save_projects_object_add', [Logger::class, 'add'], 2, 2);
add_action('save_projects_object_edit', [Logger::class, 'update'], 2, 2);
add_action('ajax_delete_projects_after_success', [Logger::class, 'delete'], 2, 2);
```

### Danh sách Hooks có sẵn

| Hook | Type | Callback nhận | Mô tả |
|------|------|---------------|--------|
| `manage_{module}_input` | filter | `(FormAdmin $form)` | Thêm field/button vào form |
| `insert_data_{module}_before_save` | filter | `($data)` | Thay đổi data trước khi lưu DB |
| `check_save_{module}_before` | filter | `($result, $request, $id)` | Validate trước khi save |
| `save_{module}_object_add` | action | `($id, $request)` | Sau khi thêm mới thành công |
| `save_{module}_object_edit` | action | `($id, $request)` | Sau khi cập nhật thành công |
| `ajax_trash_{module}_success` | action | `($id)` | Sau khi chuyển vào thùng rác |
| `ajax_restore_{module}_success` | action | `($id)` | Sau khi khôi phục |
| `ajax_delete_{module}_after_success` | action | `($id)` | Sau khi xóa vĩnh viễn |
| `manage_{module}_columns` | filter | `($columns)` | Custom cột table |
| `admin_{module}_controller_save_init` | action | | Khi form save khởi tạo |

---

## Logger (Tùy chọn)

*Đường dẫn: `app/Modules/Admin/Project/Logger.php`*

```php
<?php
namespace MyPlugin\Modules\Admin\Project;

use Admin\Supports\ActivityLogger;
use Illuminate\Support\Arr;
use SkillDo\Cms\Support\Language;

class Logger
{
    static function add($id, $request): void
    {
        $lang = Language::default();
        $title = $request->input($lang.'.title') ?: $request->input('title');

        if (!empty($title))
        {
            ActivityLogger::log('create', [
                'message' => 'Thêm dự án <b>'.$title.'</b>',
                'module'  => 'projects',
                'id'      => $id,
            ]);
        }
    }

    static function update($id, $request): void
    {
        $lang = Language::default();
        $title = $request->input($lang.'.title') ?: $request->input('title');

        if (!empty($title))
        {
            ActivityLogger::log('update', [
                'message' => 'Cập nhật dự án <b>'.$title.'</b>',
                'module'  => 'projects',
                'id'      => $id,
            ]);
        }
    }

    static function delete($id): void
    {
        $listID = Arr::wrap($id);

        if (hasItems($listID))
        {
            $objects = \MyPlugin\Models\Project::withTrashed()->whereKey($listID)->get();

            foreach ($objects as $object)
            {
                ActivityLogger::log('delete', [
                    'message' => 'Xóa dự án <b>'.$object->title.'</b>',
                    'module'  => 'projects',
                    'id'      => $object->id,
                ]);
            }
        }
    }
}
```

---

## Quy Tắc Đặt Tên

| Thành phần | Quy ước | Ví dụ |
|-----------|---------|-------|
| Module name | snake_case (số ít hoặc số nhiều) | `projects`, `brands` |
| Model class | PascalCase số ít | `Project`, `Brand` |
| Controller class | PascalCase + `Controller` | `ProjectController` |
| Form/Table class | `Form`, `Table` trong namespace Module | `MyPlugin\Modules\Admin\Project\Form` |
| Hook filter form | `manage_{module}_input` | `manage_projects_input` |
| Hook action save | `save_{module}_object_add/edit` | `save_projects_object_add` |
| Bootstrap file | kebab hoặc snake tên module | `bootstrap/projects.php` |
| View namespace | `{plugin-folder}::path` | `my-plugin::admin/projects/index` |
| Route name | `admin.{module}.{action}` | `admin.projects.index` |
| Admin page name | `{module}_add`, `{module}_edit` | `projects_add`, `projects_edit` |

## Checklist

1. ✅ Tạo Model với `$table` và `$primaryKey`
2. ✅ Tạo Route trong `routes/admin.php` (index, add, edit)
3. ✅ Tạo Form class với `fields()` và `buttons()`
4. ✅ Tạo Table class với `getColumns()`, `actionButton()`, `headerButton()`
5. ✅ Tạo Controller với `Cms::setData('module', ...)` trong constructor
6. ✅ Tạo Views (index + save) dùng `Admin::partial`
7. ✅ Đăng ký Menu Admin trong bootstrap
8. ✅ Đăng ký hooks form trong bootstrap: `manage_{module}_input`
9. ✅ (Tùy chọn) Tạo Logger cho activity log
10. ✅ View index **phải truyền** cả `name`, `module`, `model`, `table`
