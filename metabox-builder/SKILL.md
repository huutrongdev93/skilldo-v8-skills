---
name: metabox-builder
description: Hướng dẫn tạo Metabox & Metadata cho post, page, products trong SkillDo CMS v8 - đăng ký metabox, render form, lưu metadata, truy xuất frontend. Sử dụng skill này khi user yêu cầu thêm trường dữ liệu phụ, custom field, metabox, metadata cho bài viết, trang, sản phẩm. Bất kỳ khi nào nhắc đến 'thêm trường', 'metabox', 'metadata', 'custom field', 'thông tin bổ sung', 'dữ liệu phụ' cho post/page/products trong SkillDo CMS đều nên dùng skill này.
---

# Tạo Metabox & Metadata - SkillDo CMS v8

Hướng dẫn tạo Metabox (bảng phụ) để lưu thêm thông tin dưới dạng Metadata cho post, page, products mà KHÔNG cần đụng đến cột (column) trong bảng chính.

## Mục lục

1. [Tổng Quan](#1-tổng-quan)
2. [Kiến Trúc & Hooks](#2-kiến-trúc--hooks)
3. [Tạo Metabox Trong Theme](#3-tạo-metabox-trong-theme)
4. [Tạo Metabox Trong Plugin](#4-tạo-metabox-trong-plugin)
5. [Form Builder Trong Metabox](#5-form-builder-trong-metabox)
6. [Lưu Metadata](#6-lưu-metadata)
7. [Truy Xuất Metadata Frontend](#7-truy-xuất-metadata-frontend)
8. [Model Meta — Shortcut Methods](#8-model-meta--shortcut-methods-khuyến-nghị)
9. [Metadata API (Low-level)](#9-metadata-api-low-level)
10. [Áp Dụng Cho Nhiều Module](#10-áp-dụng-cho-nhiều-module)
11. [Ví Dụ Thực Tế](#11-ví-dụ-thực-tế)
12. [Quy Tắc & Lưu Ý](#12-quy-tắc--lưu-ý)
13. [Checklist](#13-checklist)

---

## 1. Tổng Quan

Metabox cho phép thêm "bảng phụ" (collapsible box) vào form tạo/sửa bài viết, trang, sản phẩm... để thu thập thêm dữ liệu mà không cần thêm cột vào bảng chính.

**Ưu điểm:**
- Mở rộng không giới hạn thông qua bảng `metadata` (gồm `object_type`, `object_id`, `meta_key`, `meta_value`)
- Không phá hủy schema bảng chính
- Tổ chức code sạch sẽ, tách biệt logic

**Quy trình tổng quát:**
1. Tạo class chứa logic `register()`, `render()`, `save()`
2. Đăng ký class vào hệ thống qua `add_action`
3. Truy xuất metadata ở frontend bằng `Model::getMeta()` hoặc `Metadata::get()`

---

## 2. Kiến Trúc & Hooks

### 2.1 Hook đăng ký Metabox

| Hook | Mô tả |
|------|--------|
| `add_meta_box` | Hệ thống gọi để thu thập tất cả metabox đã đăng ký |

### 2.2 Hook lưu dữ liệu

| Hook | Params | Mô tả |
|------|--------|--------|
| `save_{module}_object` | `($id, $request, $insertData, $dataOutside)` | Cả thêm mới lẫn cập nhật |
| `save_{module}_object_add` | `($id, $request, $insertData, $dataOutside)` | Chỉ khi thêm mới |
| `save_{module}_object_edit` | `($id, $request, $insertData, $dataOutside)` | Chỉ khi cập nhật |
| `save_object` | `($id, $module, $request, $insertData, $dataOutside)` | Bắt tất cả module |

**Trong đó `{module}` là:**
- `post` — bài viết
- `page` — trang
- `products` — sản phẩm
- `post_categories` — danh mục bài viết

### 2.3 Metabox::add() API

```php
Metabox::add($id, $title, $callback, $args = []);
```

| Param | Type | Mô tả |
|-------|------|--------|
| `$id` | string | ID duy nhất cho metabox |
| `$title` | string | Tiêu đề hiển thị của box |
| `$callback` | callable | Hàm render nội dung form. Nhận `($object)` |
| `$args` | array | Cấu hình bổ sung (xem bảng dưới) |

**Các key trong `$args`:**

| Key | Type | Default | Mô tả |
|-----|------|---------|--------|
| `module` | string\|null | `null` | Module áp dụng: `post`, `page`, `products`, `post_categories`, `null` (tất cả) |
| `position` | int | `10` | Thứ tự sắp xếp (số nhỏ = hiển thị trước) |
| `content` | string | `leftBottom` | Vị trí hiển thị: `leftTop`, `leftBottom`, `right`, `tabs` |
| `content_box` | string | `''` | CSS class tùy chỉnh cho box wrapper |

**Giải thích vị trí `content`:**

| Giá trị | Vị trí |
|---------|--------|
| `leftTop` | Cột trái, phía trên (trước nội dung chính) |
| `leftBottom` | Cột trái, phía dưới (sau nội dung chính) |
| `right` | Cột phải (sidebar) |
| `tabs` | Hiển thị dưới dạng tab riêng biệt |

### 2.4 Module con cho post_type / cate_type

Khi post có nhiều `post_type` (ví dụ: `post`, `news`, `blog`), bạn có thể chỉ áp dụng metabox cho 1 loại cụ thể:

```php
// Chỉ hiển thị cho post_type = "news"
Metabox::add('my_metabox', 'Tiêu đề', $callback, [
    'module' => 'post_news',  // post_{post_type}
]);

// Chỉ cho danh mục sản phẩm
Metabox::add('my_metabox', 'Tiêu đề', $callback, [
    'module' => 'post_categories_products_categories',  // post_categories_{cate_type}
]);
```

---

## 3. Tạo Metabox Trong Theme

### 3.1 Cấu trúc thư mục

```
theme-child/
├── app/
│   └── Custom/
│       └── {TenClass}Metabox.php    # Class metabox
└── bootstrap/
    └── theme-child.php              # Đăng ký hooks
```

### 3.2 Tạo Class Metabox

File: `theme-child/app/Custom/PostSourceMetabox.php`

```php
<?php
namespace Theme\Custom;

use SkillDo\Cms\Support\Metabox;
use SkillDo\Cms\Form\Form;
use SkillDo\Http\Request;
use SkillDo\Cms\Models\Post;

class PostSourceMetabox
{
    /**
     * Đăng ký metabox vào hệ thống.
     * Được gọi qua add_action('add_meta_box', ...) từ bootstrap.
     */
    public static function register(): void
    {
        Metabox::add(
            'source_post_metabox',            // ID duy nhất
            'Nguồn Gốc Của Bài Viết Này',     // Tiêu đề box
            [static::class, 'render'],        // Callback render
            [
                'module'   => 'post',         // Áp dụng cho module post
                'position' => 10,             // Thứ tự hiển thị
                'content'  => 'leftBottom'    // Vị trí: cột trái, phía dưới
            ]
        );
    }

    /**
     * Render nội dung Form trong Box.
     * $object là Post Data (khi Sửa) hoặc null (khi Thêm mới).
     */
    public static function render($object): void
    {
        // Dùng Model::getMeta() — cách ngắn gọn (khuyến nghị)
        $source_name = Post::getMeta($object->id, '_source_name', true);
        $source_url  = Post::getMeta($object->id, '_source_url', true);

        // Sử dụng Form Builder để tạo form chuẩn
        $form = new Form();

        echo $form->text('source_name', [
            'label'       => 'Tên Tạp chí/Báo gốc',
            'value'       => $source_name,
            'placeholder' => 'VD: VNExpress'
        ]);

        echo $form->url('source_url', [
            'label' => 'Đường dẫn link báo gốc',
            'value' => $source_url
        ]);
    }

    /**
     * Lưu dữ liệu khi người dùng nhấn Lưu.
     *
     * @param int     $post_id      ID bài viết vừa lưu
     * @param Request $request      HTTP Request chứa dữ liệu form
     * @param array   $insertData   Dữ liệu đã insert vào DB
     * @param array   $dataOutside  Dữ liệu nằm ngoài model
     */
    public static function save(int $post_id, Request $request, array $insertData, array $dataOutside): void
    {
        if ($request->has('source_name')) {
            Post::updateMeta($post_id, '_source_name', $request->input('source_name'));
            Post::updateMeta($post_id, '_source_url',  $request->input('source_url'));
        }
    }
}
```

### 3.3 Đăng ký vào Hệ thống

File: `theme-child/bootstrap/theme-child.php`

```php
<?php
use Theme\Custom\PostSourceMetabox;

// Đăng ký metabox (hiển thị form phụ trong trang tạo/sửa)
add_action('add_meta_box', [PostSourceMetabox::class, 'register']);

// Lắng nghe sự kiện lưu bài viết (cả thêm mới lẫn cập nhật)
// Hook nhận 4 tham số: $id, $request, $insertData, $dataOutside
add_action('save_post_object', [PostSourceMetabox::class, 'save'], 10, 4);
```

---

## 4. Tạo Metabox Trong Plugin

### 4.1 Cấu trúc thư mục

```
plugins/my-plugin/
├── app/
│   └── Modules/
│       └── Metabox/
│           └── ProductSpecMetabox.php
└── bootstrap/
    └── admin.php
```

### 4.2 Class Metabox trong Plugin

File: `plugins/my-plugin/app/Modules/Metabox/ProductSpecMetabox.php`

```php
<?php
namespace MyPlugin\Modules\Metabox;

use Ecommerce\Models\Product;
use SkillDo\Cms\Support\Metabox;
use SkillDo\Cms\Form\Form;
use SkillDo\Http\Request;

class ProductSpecMetabox
{
    public static function register(): void
    {
        Metabox::add(
            'product_spec_metabox',
            'Thông Số Kỹ Thuật',
            [static::class, 'render'],
            [
                'module'   => 'products',
                'position' => 15,
                'content'  => 'leftBottom'
            ]
        );
    }

    public static function render($object): void
    {
        // Dùng Model shortcut — Product tự biết table name
        $weight     = Product::getMeta($object->id, '_spec_weight', true);
        $dimensions = Product::getMeta($object->id, '_spec_dimensions', true);
        $material   = Product::getMeta($object->id, '_spec_material', true);
        $warranty   = Product::getMeta($object->id, '_spec_warranty', true);

        $form = new Form();

        echo $form->text('spec_weight', [
            'label'       => 'Cân nặng',
            'value'       => $weight,
            'placeholder' => 'VD: 500g',
            'start'       => 6,
        ]);

        echo $form->text('spec_dimensions', [
            'label'       => 'Kích thước',
            'value'       => $dimensions,
            'placeholder' => 'VD: 20x30x10 cm',
            'start'       => 6,
        ]);

        echo $form->text('spec_material', [
            'label' => 'Chất liệu',
            'value' => $material,
        ]);

        echo $form->text('spec_warranty', [
            'label'       => 'Bảo hành',
            'value'       => $warranty,
            'placeholder' => 'VD: 12 tháng',
        ]);
    }

    public static function save(int $product_id, Request $request, array $insertData, array $dataOutside): void
    {
        if ($request->has('spec_weight')) {
            Product::updateMeta($product_id, '_spec_weight',     $request->input('spec_weight'));
            Product::updateMeta($product_id, '_spec_dimensions', $request->input('spec_dimensions'));
            Product::updateMeta($product_id, '_spec_material',   $request->input('spec_material'));
            Product::updateMeta($product_id, '_spec_warranty',   $request->input('spec_warranty'));
        }
    }
}
```

### 4.3 Đăng ký trong Bootstrap

File: `plugins/my-plugin/bootstrap/admin.php`

```php
<?php
use MyPlugin\Modules\Metabox\ProductSpecMetabox;

// Đăng ký metabox
add_action('add_meta_box', [ProductSpecMetabox::class, 'register']);

// Lưu metadata khi save sản phẩm
add_action('save_products_object', [ProductSpecMetabox::class, 'save'], 10, 4);
```

---

## 5. Form Builder Trong Metabox

Trong phương thức `render()`, sử dụng `SkillDo\Cms\Form\Form` để tạo các field. Tham khảo skill **form-builder** để biết đầy đủ các loại field.

### 5.1 Các field thường dùng trong metabox

```php
public static function render($object): void
{
    // Lấy giá trị cũ
    $value = Metadata::get('{module}', $object->id, '_meta_key', '');

    $form = new Form();

    // Text input
    echo $form->text('field_name', [
        'label' => 'Label',
        'value' => $value,
        'placeholder' => 'Gợi ý...',
    ]);

    // Textarea
    echo $form->textarea('field_desc', [
        'label' => 'Mô tả',
        'value' => Metadata::get('{module}', $object->id, '_field_desc', ''),
    ]);

    // Select dropdown
    echo $form->select('field_type', [
        'label' => 'Loại',
        'value' => Metadata::get('{module}', $object->id, '_field_type', ''),
    ])->options([
        'type1' => 'Loại 1',
        'type2' => 'Loại 2',
    ]);

    // Switch on/off
    echo $form->switch('field_featured', [
        'label' => 'Nổi bật',
        'value' => Metadata::get('{module}', $object->id, '_field_featured', 0),
    ]);

    // Number
    echo $form->number('field_order', [
        'label' => 'Thứ tự',
        'value' => Metadata::get('{module}', $object->id, '_field_order', 0),
        'min'   => 0,
    ]);

    // Image
    echo $form->image('field_banner', [
        'label' => 'Banner',
        'value' => Metadata::get('{module}', $object->id, '_field_banner', ''),
    ]);

    // Color
    echo $form->color('field_color', [
        'label' => 'Màu sắc',
        'value' => Metadata::get('{module}', $object->id, '_field_color', ''),
    ]);

    // Date
    echo $form->date('field_date', [
        'label' => 'Ngày',
        'value' => Metadata::get('{module}', $object->id, '_field_date', ''),
    ]);

    // URL
    echo $form->url('field_link', [
        'label' => 'Đường dẫn',
        'value' => Metadata::get('{module}', $object->id, '_field_link', ''),
    ]);

    // WYSIWYG editor
    echo $form->wysiwyg('field_content', [
        'label' => 'Nội dung',
        'value' => Metadata::get('{module}', $object->id, '_field_content', ''),
    ]);

    // Layout 2 cột (start = số cột, tổng 12)
    echo $form->text('col1', ['label' => 'Cột 1', 'start' => 6, 'value' => '']);
    echo $form->text('col2', ['label' => 'Cột 2', 'start' => 6, 'value' => '']);
}
```

### 5.2 Repeater trong Metabox

```php
public static function render($object): void
{
    $items = Metadata::get('{module}', $object->id, '_faq_items', '');

    $form = new Form();

    echo $form->repeater('faq_items', [
        'label'  => 'Câu hỏi thường gặp',
        'value'  => $items,
        'fields' => function (Form $form) {
            $form->text('question', ['label' => 'Câu hỏi']);
            $form->textarea('answer', ['label' => 'Trả lời']);
        }
    ]);
}
```

---

## 6. Lưu Metadata

### 6.1 Mẫu lưu cơ bản

```php
public static function save(int $id, Request $request, array $insertData, array $dataOutside): void
{
    // Cách 1: Dùng Model shortcut (khuyến nghị)
    if ($request->has('field_name')) {
        Post::updateMeta($id, '_meta_key', $request->input('field_name'));
    }

    // Cách 2: Dùng Metadata class trực tiếp
    if ($request->has('field_name')) {
        Metadata::update('post', $id, '_meta_key', $request->input('field_name'));
    }
}
```

### 6.2 Lưu nhiều field

```php
public static function save(int $id, Request $request, array $insertData, array $dataOutside): void
{
    $fields = [
        'field_name'     => '_meta_name',
        'field_desc'     => '_meta_desc',
        'field_type'     => '_meta_type',
        'field_featured' => '_meta_featured',
    ];

    foreach ($fields as $requestKey => $metaKey) {
        if ($request->has($requestKey)) {
            Post::updateMeta($id, $metaKey, $request->input($requestKey));
        }
    }
}
```

### 6.3 Lưu dữ liệu phức tạp (array/object)

Metadata tự động `json_encode` khi value là array hoặc object:

```php
public static function save(int $id, Request $request, array $insertData, array $dataOutside): void
{
    // Lưu repeater data (tự động serialize)
    if ($request->has('faq_items')) {
        Post::updateMeta($id, '_faq_items', $request->input('faq_items'));
    }

    // Lưu array thủ công
    $specs = [
        'weight'     => $request->input('spec_weight'),
        'dimensions' => $request->input('spec_dimensions'),
        'material'   => $request->input('spec_material'),
    ];
    Product::updateMeta($id, '_product_specs', $specs);
}
```

### 6.4 Hook chỉ khi thêm mới hoặc chỉ khi cập nhật

```php
// Chỉ khi thêm mới
add_action('save_post_object_add', [MyMetabox::class, 'onAdd'], 10, 4);

// Chỉ khi cập nhật
add_action('save_post_object_edit', [MyMetabox::class, 'onEdit'], 10, 4);

// Bắt tất cả module
add_action('save_object', [MyMetabox::class, 'onSaveAll'], 10, 5);
// Callback: save($id, $module, $request, $insertData, $dataOutside)
```

---

## 7. Truy Xuất Metadata Frontend

### 7.1 Trong file View Blade

```php
@php
    use SkillDo\Cms\Models\Post;

    // Cách 1: Model shortcut (khuyến nghị)
    $source_name = Post::getMeta($post->id, '_source_name', true);
    $source_url  = Post::getMeta($post->id, '_source_url', true);

    // Cách 2: Metadata class trực tiếp
    // $source_name = Metadata::get('post', $post->id, '_source_name', true);
@endphp

@if($source_name && $source_url)
    <div class="source-credit" style="padding:10px; background:#f0f0f0;">
        <p><strong>Bản quyền:</strong> <a href="{{ $source_url }}" target="_blank">{{ $source_name }}</a></p>
    </div>
@endif
```

### 7.2 Trong Controller hoặc PHP

```php
use Ecommerce\Models\Product;
use SkillDo\Cms\Models\Post;

// Lấy 1 giá trị cụ thể — dùng Model shortcut
$value = Product::getMeta($productId, '_spec_weight', true);

// Lấy tất cả metadata của object
$allMeta = Post::getMeta($postId);
// Trả về object: $allMeta->_source_name, $allMeta->_source_url

// Lấy 1 bản ghi duy nhất (single = true)
$single = Post::getMeta($postId, '_source_name', true);
```

### 7.3 Trong Element (Page Builder widget)

```php
public function widget(): void
{
    $postId = $this->options->postId ?? 0;
    
    if ($postId) {
        $specWeight = Product::getMeta($postId, '_spec_weight', true);
    }
    
    Theme::view($this->getDir().'views/view', [
        'options'    => $this->options,
        'specWeight' => $specWeight ?? '',
    ]);
}
```

---

## 8. Model Meta — Shortcut Methods (Khuyến nghị)

Tất cả Model trong SkillDo CMS đều kế thừa trait `ModelMeta` từ class cha `Model`. Điều này nghĩa là mọi model (Post, Page, Product, custom model...) đều có sẵn các method thao tác metadata mà **không cần truyền `$object_type`** — model tự biết table name.

### 8.1 Các method có sẵn trên Model

| Method | Signature | Mô tả |
|--------|-----------|--------|
| `Model::getMeta()` | `getMeta($objectId, $key = null, $single = true)` | Lấy metadata |
| `Model::addMeta()` | `addMeta($objectId, $key, $value)` | Thêm metadata mới |
| `Model::updateMeta()` | `updateMeta($objectId, $key, $value)` | Cập nhật (tự tạo nếu chưa có) |
| `Model::deleteMeta()` | `deleteMeta($objectId, $key, $value)` | Xóa metadata |

### 8.2 So sánh 2 cách gọi

```php
use SkillDo\Cms\Models\Post;
use SkillDo\Cms\Models\Page;
use Ecommerce\Models\Product;

// ✅ Model shortcut (khuyến nghị) — ngắn gọn, type-safe
$value = Post::getMeta($postId, '_source_name', true);
Post::updateMeta($postId, '_source_name', 'VNExpress');
Post::deleteMeta($postId, '_source_name');

// Tương đương với Metadata class:
$value = Metadata::get('post', $postId, '_source_name', true);
Metadata::update('post', $postId, '_source_name', 'VNExpress');
Metadata::delete('post', $postId, '_source_name');

// Product
$weight = Product::getMeta($productId, '_spec_weight', true);
Product::updateMeta($productId, '_spec_weight', '500g');

// Page  
$hideHeader = Page::getMeta($pageId, '_hide_header', true);
Page::updateMeta($pageId, '_hide_header', 1);

// Custom model trong plugin
$meta = MyModel::getMeta($id, '_custom_field', true);
MyModel::updateMeta($id, '_custom_field', 'value');
```

### 8.3 Ưu điểm Model shortcut

- **Không cần nhớ tên table**: `Product::getMeta(...)` tự biết table là `products`
- **Type-safe**: IDE có thể autocomplete và kiểm tra class
- **Ngắn gọn**: bỏ qua tham số `$object_type`
- **Nhất quán**: dùng chung pattern với các method khác của Model (`find`, `create`, `where`...)

### 8.4 Khi nào dùng Metadata:: trực tiếp

- Khi metabox áp dụng cho **tất cả module** (`module => null`) và dùng hook `save_object` — lúc này `$module` là biến động
- Khi làm việc với module không có Model class riêng

---

## 9. Metadata API (Low-level)

### 9.1 Các phương thức

| Method | Signature | Mô tả |
|--------|-----------|--------|
| `Metadata::get()` | `get($type, $id, $key, $single)` | Lấy metadata |
| `Metadata::add()` | `add($type, $id, $key, $value)` | Thêm metadata mới |
| `Metadata::update()` | `update($type, $id, $key, $value)` | Cập nhật (tự tạo nếu chưa có) |
| `Metadata::delete()` | `delete($type, $id, $key, $value, $all)` | Xóa metadata |
| `Metadata::count()` | `count($type, $args)` | Đếm số metadata |
| `Metadata::deleteAll()` | `deleteAll($type, $key)` | Xóa tất cả theo type |
| `Metadata::deleteByMid()` | `deleteByMid($type, $mid)` | Xóa theo object_id |

### 9.2 Chi tiết Metadata::get()

```php
Metadata::get($object_type, $object_id, $meta_key = '', $single = false);
```

| Param | Type | Mô tả |
|-------|------|--------|
| `$object_type` | string | Loại đối tượng: `post`, `page`, `products`, `menu`... |
| `$object_id` | int | ID của đối tượng |
| `$meta_key` | string | Key metadata cần lấy. Rỗng = lấy tất cả |
| `$single` | bool\|mixed | `true` = lấy bản ghi đầu tiên. Giá trị khác = default khi không tìm thấy |

**Trả về:**
- Khi `$single = true` hoặc có `$meta_key`: trả về giá trị (string, array, object tùy vào data đã lưu)
- Khi `$meta_key` rỗng và `$single = false`: trả về object chứa tất cả key-value
- Tự động `json_decode` nếu value là JSON
- Tự động `unserialize` nếu value là serialized

### 9.3 Chi tiết Metadata::update()

```php
Metadata::update($object_type, $object_id, $meta_key, $meta_value);
```

- Nếu `$meta_key` **chưa tồn tại** → tự động gọi `Metadata::add()` để tạo mới
- Nếu `$meta_value` là array/object → tự động `json_encode`
- Trả về `true` khi thành công, `false` khi thất bại

### 9.4 Chi tiết Metadata::delete()

```php
Metadata::delete($object_type, $object_id, $meta_key = '', $meta_value = '', $delete_all = false);
```

- `$meta_key` rỗng: xóa tất cả metadata của object
- `$delete_all = true`: xóa tất cả object cùng type có `$meta_key`

---

## 10. Áp Dụng Cho Nhiều Module

### 9.1 Một metabox cho nhiều module

Đặt `module => null` để metabox hiển thị ở tất cả module:

```php
public static function register(): void
{
    Metabox::add('seo_extra', 'SEO Bổ Sung', [static::class, 'render'], [
        'module'   => null,        // Hiển thị ở mọi module
        'position' => 99,
        'content'  => 'leftBottom'
    ]);
}
```

Khi lưu, dùng hook `save_object` (bắt tất cả module):

```php
// Bootstrap
add_action('save_object', [SeoExtraMetabox::class, 'save'], 10, 5);

// Trong class
public static function save(int $id, string $module, Request $request, array $insertData, array $dataOutside): void
{
    if ($request->has('seo_canonical')) {
        Metadata::update($module, $id, '_seo_canonical', $request->input('seo_canonical'));
    }
}
```

### 9.2 Đăng ký riêng cho từng module

```php
public static function register(): void
{
    // Metabox cho Post
    Metabox::add('post_extra', 'Thông Tin Bổ Sung', [static::class, 'renderPost'], [
        'module' => 'post',
        'content' => 'leftBottom'
    ]);

    // Metabox cho Page
    Metabox::add('page_extra', 'Cấu Hình Trang', [static::class, 'renderPage'], [
        'module' => 'page',
        'content' => 'right'
    ]);

    // Metabox cho Products
    Metabox::add('product_extra', 'Thông Số Sản Phẩm', [static::class, 'renderProduct'], [
        'module' => 'products',
        'content' => 'leftBottom'
    ]);
}
```

Đăng ký hook save tương ứng:

```php
add_action('save_post_object', [MyMetabox::class, 'savePost'], 10, 4);
add_action('save_page_object', [MyMetabox::class, 'savePage'], 10, 4);
add_action('save_products_object', [MyMetabox::class, 'saveProduct'], 10, 4);
```

---

## 11. Ví Dụ Thực Tế

### 10.1 Metabox nguồn bài viết (Post)

```php
<?php
namespace Theme\Custom;

use SkillDo\Cms\Models\Post;
use SkillDo\Cms\Support\Metabox;
use SkillDo\Cms\Form\Form;
use SkillDo\Http\Request;

class PostSourceMetabox
{
    public static function register(): void
    {
        Metabox::add('source_post_metabox', 'Nguồn Gốc Bài Viết', [static::class, 'render'], [
            'module'   => 'post',
            'position' => 10,
            'content'  => 'leftBottom'
        ]);
    }

    public static function render($object): void
    {
        $source_name = Post::getMeta($object->id, '_source_name', true);
        $source_url  = Post::getMeta($object->id, '_source_url', true);

        $form = new Form();

        echo $form->text('source_name', [
            'label' => 'Tên báo gốc', 'value' => $source_name,
            'placeholder' => 'VD: VNExpress', 'start' => 6,
        ]);
        echo $form->url('source_url', [
            'label' => 'Link báo gốc', 'value' => $source_url, 'start' => 6,
        ]);
    }

    public static function save(int $id, Request $request, array $insertData, array $dataOutside): void
    {
        if ($request->has('source_name')) {
            Post::updateMeta($id, '_source_name', $request->input('source_name'));
            Post::updateMeta($id, '_source_url',  $request->input('source_url'));
        }
    }
}
```

### 11.2 Metabox cấu hình trang (Page)

```php
<?php
namespace Theme\Custom;

use SkillDo\Cms\Models\Page;
use SkillDo\Cms\Support\Metabox;
use SkillDo\Cms\Form\Form;
use SkillDo\Http\Request;

class PageConfigMetabox
{
    public static function register(): void
    {
        Metabox::add('page_config', 'Cấu Hình Trang', [static::class, 'render'], [
            'module'   => 'page',
            'position' => 5,
            'content'  => 'right'
        ]);
    }

    public static function render($object): void
    {
        $hide_header = Page::getMeta($object->id, '_hide_header', true);
        $hide_footer = Page::getMeta($object->id, '_hide_footer', true);
        $bg_color    = Page::getMeta($object->id, '_bg_color', true);

        $form = new Form();

        echo $form->switch('hide_header', [
            'label' => 'Ẩn Header', 'value' => $hide_header,
        ]);
        echo $form->switch('hide_footer', [
            'label' => 'Ẩn Footer', 'value' => $hide_footer,
        ]);
        echo $form->color('bg_color', [
            'label' => 'Màu nền trang', 'value' => $bg_color,
        ]);
    }

    public static function save(int $id, Request $request, array $insertData, array $dataOutside): void
    {
        Page::updateMeta($id, '_hide_header', $request->input('hide_header', 0));
        Page::updateMeta($id, '_hide_footer', $request->input('hide_footer', 0));

        if ($request->has('bg_color')) {
            Page::updateMeta($id, '_bg_color', $request->input('bg_color'));
        }
    }
}
```

### 11.3 Metabox thông số sản phẩm (Products)

```php
<?php
namespace MyPlugin\Modules\Metabox;

use Ecommerce\Models\Product;
use SkillDo\Cms\Support\Metabox;
use SkillDo\Cms\Form\Form;
use SkillDo\Http\Request;

class ProductSpecMetabox
{
    public static function register(): void
    {
        Metabox::add('product_specs', 'Thông Số Kỹ Thuật', [static::class, 'render'], [
            'module'   => 'products',
            'position' => 15,
            'content'  => 'leftBottom'
        ]);
    }

    public static function render($object): void
    {
        $form = new Form();

        $fields = [
            'spec_weight'     => ['label' => 'Cân nặng',    'placeholder' => 'VD: 500g'],
            'spec_dimensions' => ['label' => 'Kích thước',   'placeholder' => 'VD: 20x30x10 cm'],
            'spec_material'   => ['label' => 'Chất liệu',    'placeholder' => 'VD: Nhựa ABS'],
            'spec_origin'     => ['label' => 'Xuất xứ',      'placeholder' => 'VD: Việt Nam'],
            'spec_warranty'   => ['label' => 'Bảo hành',     'placeholder' => 'VD: 12 tháng'],
        ];

        foreach ($fields as $key => $args) {
            $value = Product::getMeta($object->id, '_'.$key, true);
            echo $form->text($key, array_merge($args, [
                'value' => $value,
                'start' => 6,
            ]));
        }
    }

    public static function save(int $id, Request $request, array $insertData, array $dataOutside): void
    {
        $fields = ['spec_weight', 'spec_dimensions', 'spec_material', 'spec_origin', 'spec_warranty'];

        foreach ($fields as $field) {
            if ($request->has($field)) {
                Product::updateMeta($id, '_'.$field, $request->input($field));
            }
        }
    }
}
```

### 11.4 Metabox vị trí sidebar phải

```php
Metabox::add('product_badge', 'Nhãn Sản Phẩm', [static::class, 'render'], [
    'module'   => 'products',
    'position' => 5,
    'content'  => 'right'      // Hiển thị ở sidebar phải
]);
```

### 11.5 Metabox dạng Tab

```php
Metabox::add('product_video', 'Video Sản Phẩm', [static::class, 'render'], [
    'module'   => 'products',
    'position' => 20,
    'content'  => 'tabs'       // Hiển thị dưới dạng tab riêng biệt
]);
```

---

## 12. Quy Tắc & Lưu Ý

### 12.1 Quy tắc đặt tên

| Thành phần | Quy ước | Ví dụ |
|-----------|---------|-------|
| Metabox ID | snake_case, mô tả rõ ràng | `source_post_metabox`, `product_specs` |
| Meta key | Bắt đầu bằng `_` = ẩn ở UI | `_source_name`, `_spec_weight` |
| Meta key | Không bắt đầu `_` = Custom Field UI | `source_name`, `spec_weight` |
| Form field name | snake_case, không có `_` đầu | `source_name`, `spec_weight` |
| Class name | PascalCase + `Metabox` suffix | `PostSourceMetabox`, `ProductSpecMetabox` |

### 12.2 Lưu ý quan trọng

1. **Meta key bắt đầu `_` (underscore)** là ẩn ở giao diện Custom Field mặc định — chỉ truy cập qua code
2. **Meta key không có `_`** sẽ hiển thị trong danh sách Custom Field ở admin
3. **`Model::updateMeta()` / `Metadata::update()` tự động tạo mới** nếu key chưa tồn tại (không cần gọi `add()` trước)
4. **Array/Object tự động JSON encode** khi lưu và JSON decode khi đọc
5. **Tên field trong form (`source_name`)** và tên meta key (`_source_name`) nên khác nhau để tránh xung đột
6. **Hook `save_{module}_object`** nhận 4 tham số — phải khai báo `10, 4` khi `add_action`
7. **Hook `save_object`** nhận 5 tham số (thêm `$module`) — phải khai báo `10, 5`
8. Metadata được **cache tự động** — không cần lo về performance khi gọi `getMeta()` nhiều lần
9. **Ưu tiên dùng Model::getMeta/updateMeta** thay vì Metadata:: trực tiếp để code ngắn gọn hơn

### 12.3 Namespace Import

```php
// Metabox class (đăng ký bảng phụ)
use SkillDo\Cms\Support\Metabox;

// Form builder (tạo field trong render)
use SkillDo\Cms\Form\Form;

// Request (nhận dữ liệu form khi save)
use SkillDo\Http\Request;

// Model class (thao tác metadata — khuyến nghị)
use SkillDo\Cms\Models\Post;
use SkillDo\Cms\Models\Page;
use Ecommerce\Models\Product;

// Hoặc Metadata facade (low-level)
use Metadata;
```

---

## 13. Checklist

### Tạo Metabox mới

1. ✅ Tạo class với 3 phương thức static: `register()`, `render()`, `save()`
2. ✅ Trong `register()`: gọi `Metabox::add()` với ID, title, callback, args (module, position, content)
3. ✅ Trong `render($object)`: dùng `Model::getMeta()` lấy giá trị cũ, `Form` builder tạo field
4. ✅ Trong `save($id, $request, ...)`: dùng `Model::updateMeta()` lưu từ `$request->input()`
5. ✅ Đăng ký hook `add_meta_box` → `register()`
6. ✅ Đăng ký hook `save_{module}_object` → `save()` với params đúng (10, 4)
7. ✅ Kiểm tra `$request->has()` trước khi lưu để tránh ghi đè rỗng
8. ✅ Dùng prefix `_` cho meta_key nếu muốn ẩn khỏi Custom Field UI
9. ✅ Truy xuất ở frontend bằng `Model::getMeta()` hoặc `Metadata::get()`

### Theme

- Đặt class trong `theme-child/app/Custom/`
- Đăng ký hooks trong `theme-child/bootstrap/theme-child.php`

### Plugin

- Đặt class trong `plugins/{plugin}/app/Modules/Metabox/`
- Đăng ký hooks trong `plugins/{plugin}/bootstrap/admin.php`
