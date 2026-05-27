---
name: plugin-builder
description: Hướng dẫn tạo Plugin hoàn chỉnh cho SkillDo CMS v8 - cấu trúc thư mục, plugin.json, Main Class, ServiceProvider, Routes, Bootstrap, Assets, Ajax và Lifecycle. Sử dụng skill này khi user yêu cầu tạo plugin mới, migrate plugin, hoặc thêm tính năng vào plugin hiện có. Bất kỳ khi nào nhắc đến 'tạo plugin', 'plugin mới', 'phát triển plugin', 'scaffold plugin' cho SkillDo CMS đều nên dùng skill này.
---

# Tạo Plugin - SkillDo CMS v8

Hướng dẫn tạo plugin hoàn chỉnh từ A đến Z cho SkillDo CMS v8. Plugin nằm trong thư mục `plugins/` của dự án.

## Mục lục

1. [Cấu Trúc Thư Mục](#1-cấu-trúc-thư-mục)
2. [plugin.json — Manifest](#2-pluginjson--manifest)
3. [Main Class — Entry Point](#3-main-class--entry-point)
4. [ServiceProvider](#4-serviceprovider)
5. [Routes](#5-routes)
6. [Bootstrap — Hook Registration](#6-bootstrap--hook-registration)
7. [Assets — CSS & JS](#7-assets--css--js)
8. [Ajax Handlers](#8-ajax-handlers)
9. [Lifecycle & Hooks hệ thống](#9-lifecycle--hooks-hệ-thống)
10. [Quy tắc đặt tên](#10-quy-tắc-đặt-tên)
11. [Checklist](#11-checklist)
12. [Templates theo cấp độ](#12-templates-theo-cấp-độ)

---

## 1. Cấu Trúc Thư Mục

Tên thư mục plugin: **kebab-case** (VD: `my-plugin`, `skd-seo`).

```
plugins/my-plugin/
│
├── plugin.json                    # ① Manifest (BẮT BUỘC)
├── my-plugin.php                  # ② Main Class — hoặc index.php (BẮT BUỘC)
│
├── bootstrap/                     # ③ Bootstrap: đăng ký hooks, auto-loaded
│   ├── admin.php                  #    Hooks cho Admin
│   ├── web.php                    #    Hooks cho Frontend
│   └── ajax.php                   #    Đăng ký Ajax handlers
│
├── app/                           # ④ Source Code PHP (namespace: MyPlugin\*)
│   ├── Ajax/
│   │   ├── Admin/                 #    Ajax::admin(...)
│   │   └── Web/                   #    Ajax::client(...) / Ajax::login(...)
│   ├── Controllers/
│   │   ├── Admin/                 #    Admin HTTP Controllers
│   │   └── Web/                   #    Frontend HTTP Controllers
│   ├── Models/                    #    Eloquent Models
│   ├── Providers/
│   │   └── PluginServiceProvider.php
│   ├── Services/
│   │   ├── Activator.php          #    Logic kích hoạt plugin
│   │   ├── Deactivator.php        #    Logic gỡ cài đặt
│   │   └── AdminService.php       #    Menu, Assets, Breadcrumb
│   ├── Supports/                  #    Helper classes
│   ├── Modules/                   #    Feature modules (Tables, Forms, Settings...)
│   │   └── Admin/
│   └── Notifications/             #    Notification classes
│
├── routes/                        # ⑤ Routes (CMS tự quét)
│   ├── admin.php
│   ├── web.php
│   └── api.php
│
├── views/                         # ⑥ Blade Views
│   ├── admin/
│   └── web/
│
├── config/                        # ⑦ Config (truy cập: config('my-plugin::key'))
│   └── config.php
│
├── language/                      # ⑧ Translations (truy cập: trans('my-plugin::file.key'))
│   ├── vi/messages.php
│   └── en/messages.php
│
└── assets/                        # ⑨ Static Assets
    ├── css/
    ├── js/
    └── thumb.png
```

**Quy tắc tên file Main Class:**
- File phải đặt **ngang hàng plugin.json** (root plugin)
- Tên file = tên thư mục plugin (VD: `my-plugin.php`) hoặc `index.php`
- Cả hai cách đều hợp lệ, nhưng tên trùng thư mục là convention chính thức

---

## 2. plugin.json — Manifest

File quan trọng nhất — CMS dựa vào đây để nhận diện plugin.

### 2.1 Template tối thiểu

```json
{
    "name": "My Plugin",
    "class": "MyPlugin",
    "version": "1.0.0",
    "description": "Mô tả plugin.",
    "author": "Author Name",
    "autoload": {
        "alias": "MyPlugin"
    }
}
```

### 2.2 Template đầy đủ

```json
{
    "name": "My Plugin",
    "class": "MyPlugin",
    "url": "https://sikido.vn",
    "description": "Mô tả plugin.",
    "author": "Author Name",
    "version": "1.0.0",
    "thumb": "assets/thumb.png",
    "providers": [
        "MyPlugin\\Providers\\PluginServiceProvider"
    ],
    "autoload": {
        "alias": "MyPlugin",
        "psr-4": {
            "MyPlugin\\Notifications": "app\\Notifications"
        },
        "files": [
            "bootstrap/ajax.php"
        ]
    },
    "aliases": {
        "controller": {
            "MyPlugin\\Controllers\\Admin\\ItemController": "items"
        },
        "hooks": {},
        "middlewares": {}
    },
    "middlewares": {
        "groups": {
            "web": []
        }
    },
    "cms": {
        "form": {
            "popover": {},
            "fields": {}
        }
    }
}
```

### 2.3 Giải thích các key

| Key | Bắt buộc | Mô tả |
|-----|----------|-------|
| `name` | ✅ | Tên hiển thị plugin |
| `class` | ✅ | Tên class Main (trùng class trong file main) |
| `version` | ❌ | Phiên bản hiện tại |
| `description` | ❌ | Mô tả plugin |
| `author` | ❌ | Tác giả |
| `url` | ❌ | URL trang web |
| `thumb` | ❌ | Ảnh đại diện (tương đối từ root plugin) |
| `providers` | ❌ | Danh sách ServiceProvider classes |
| `autoload` | ❌ | Cấu hình PSR-4 autoloading |
| `aliases` | ❌ | Alias cho controller, hooks, middlewares |
| `middlewares` | ❌ | Middleware groups |
| `cms` | ❌ | Đăng ký custom form fields/popovers |

### 2.4 Autoload — hai cách khai báo

**Cách 1: Dùng `alias` (khuyến nghị)** — CMS tự map namespace vào các thư mục chuẩn:

```json
{
    "autoload": {
        "alias": "MyPlugin"
    }
}
```

Tự động map: `MyPlugin\Ajax` → `app/Ajax`, `MyPlugin\Controllers` → `app/Controllers`, `MyPlugin\Models` → `app/Models`, `MyPlugin\Modules` → `app/Modules`, `MyPlugin\Providers` → `app/Providers`, `MyPlugin\Services` → `app/Services`, `MyPlugin\Supports` → `app/Supports`, `MyPlugin\Cms` → `app/Cms`, `MyPlugin\Middlewares` → `app/Middlewares`.

**Kết hợp cả hai** khi cần thêm namespace ngoài chuẩn:

```json
{
    "autoload": {
        "alias": "MyPlugin",
        "psr-4": {
            "MyPlugin\\Notifications": "app\\Notifications",
            "MyPlugin\\Status": "app\\Status"
        }
    }
}
```

### 2.5 Aliases — Controller, Hooks, Middleware

**Controller alias** → tạo Page Key cho `Admin::isPage()`:

```json
{
    "aliases": {
        "controller": {
            "MyPlugin\\Controllers\\Admin\\ItemController": "items"
        }
    }
}
```
Quy tắc Page Key: `{alias}_{method}` → `items_index`, `items_add`, `items_edit`.

**Hook alias** → tên ngắn cho static method:

```json
{
    "aliases": {
        "hooks": {
            "page_items_index": "MyPlugin\\Template\\ItemsIndex::index"
        }
    }
}
```

**Middleware alias** → tên ngắn dùng trong routes:

```json
{
    "aliases": {
        "middlewares": {
            "myplugin.auth": "MyPlugin\\Middlewares\\MyAuth"
        }
    }
}
```

---

## 3. Main Class — Entry Point

File nằm ở root plugin, tên trùng thư mục hoặc `index.php`.

### 3.1 Plugin đơn giản (không cần database)

```php
<?php
// plugins/my-plugin/my-plugin.php

class MyPlugin
{
    private string $name = 'my_plugin';

    public function active(): void
    {
        // Không cần tạo bảng database
    }

    public function uninstall(): void
    {
        // Xóa options nếu có
        \SkillDo\Cms\Support\Option::delete('my_plugin_config');
    }
}
```

### 3.2 Plugin có database (delegate sang Service)

```php
<?php
// plugins/my-plugin/my-plugin.php  (hoặc index.php)

use MyPlugin\Services\Activator;
use MyPlugin\Services\Deactivator;

class MyPlugin
{
    public function active(): void
    {
        Activator::activate();
    }

    public function uninstall(): void
    {
        Deactivator::uninstall();
    }
}
```

### 3.3 Activator Service

```php
<?php
namespace MyPlugin\Services;

use Illuminate\Support\Facades\DB;

class Activator
{
    public static function activate(): void
    {
        self::createTable();
    }

    public static function createTable(): void
    {
        if (!schema()->hasTable('my_items'))
        {
            schema()->create('my_items', function ($table) {
                $table->increments('id');
                $table->string('title', 255)->collate('utf8mb4_unicode_ci');
                $table->text('content')->collate('utf8mb4_unicode_ci')->nullable();
                $table->string('image', 255)->collate('utf8mb4_unicode_ci')->nullable();
                $table->integer('order')->default(0);
                $table->dateTime('created')->default(DB::raw('CURRENT_TIMESTAMP'));
                $table->dateTime('updated')->nullable();
            });
        }
    }
}
```

### 3.4 Deactivator Service

```php
<?php
namespace MyPlugin\Services;

class Deactivator
{
    public static function uninstall(): void
    {
        schema()->dropIfExists('my_items');
    }
}
```

---

## 4. ServiceProvider

Đăng ký trong `plugin.json` → `providers`. CMS gọi `register()` rồi `boot()` khi khởi động.

```php
<?php
namespace MyPlugin\Providers;

use SkillDo\ServiceProvider;

class PluginServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Đăng ký bindings, alias
    }

    public function boot(): void
    {
        // Đăng ký hooks, events
        // Thường để trống nếu dùng bootstrap files
    }
}
```

**Khi nào dùng ServiceProvider vs Bootstrap files:**
- **ServiceProvider**: đăng ký alias (`AliasLoader`), bindings, config loading
- **Bootstrap files**: đăng ký hooks (`add_action`, `add_filter`), ajax handlers — linh hoạt hơn, tách theo context (admin/web)

---

## 5. Routes

CMS **tự động quét** thư mục `routes/` trong plugin. Chỉ cần tạo file đúng tên.

### 5.1 Admin Routes (`routes/admin.php`)

```php
<?php
use SkillDo\Support\Facades\Route;
use MyPlugin\Controllers\Admin\ItemController;

Route::middleware('auth:admin')->prefix('admin/items')->group(function () {
    $controller = ItemController::class;

    Route::match(['get','post'], '/',          $controller.'@index')->name('admin.items.index');
    Route::match(['get','post'], '/add',       $controller.'@add')->name('admin.items.add');
    Route::match(['get','post'], '/edit/{id}',  $controller.'@edit')->where('id', '[0-9]+')->name('admin.items.edit');
});
```

### 5.2 Frontend Routes (`routes/web.php`)

```php
<?php
use SkillDo\Support\Facades\Route;
use MyPlugin\Controllers\Web\ItemController;

Route::get('items', [ItemController::class, 'index']);
Route::get('items/{slug}', [ItemController::class, 'detail']);
```

### 5.3 API Routes (`routes/api.php`)

```php
<?php
use SkillDo\Support\Facades\Route;

Route::prefix('api/v1/items')->group(function () {
    Route::get('/', 'MyPlugin\Controllers\Api\ItemController@list');
});
```

---

## 6. Bootstrap — Hook Registration

Files trong `bootstrap/` tự động nạp.

### 6.1 Admin hooks (`bootstrap/admin.php`)

```php
<?php
use MyPlugin\Services\AdminService;
use MyPlugin\Modules\Admin\ItemForm;

// Đăng ký menu admin
add_action('admin_navigation', [AdminService::class, 'navigation'], 20);

// Đăng ký admin assets
add_action('admin_assets', [AdminService::class, 'assets'], 50);

// Đăng ký form hooks cho module CRUD (nếu dùng module-builder)
add_filter('manage_items_input', [ItemForm::class, 'fields']);
add_filter('manage_items_input', [ItemForm::class, 'buttons']);
```

### 6.2 Frontend hooks (`bootstrap/web.php`)

```php
<?php
use MyPlugin\Services\AssetsService;

// Đăng ký frontend assets
add_action('theme_custom_assets', [AssetsService::class, 'web'], 10, 2);

// Đăng ký CSS variables
add_filter('theme_head_style_variable', [AssetsService::class, 'webVariable'], 30);
```

### 6.3 Ajax handlers (`bootstrap/ajax.php`)

```php
<?php
use SkillDo\Cms\Support\Ajax;

// Admin-only ajax
Ajax::admin('MyPlugin\Ajax\Admin\ItemAjax::load');
Ajax::admin('MyPlugin\Ajax\Admin\ItemAjax::save');

// Client ajax (không cần đăng nhập)
Ajax::client('MyPlugin\Ajax\Web\PublicAjax::search');

// Login-required ajax
Ajax::login('MyPlugin\Ajax\Web\UserAjax::submit');
```

### 6.4 System Settings (`bootstrap/admin.php`)

Đăng ký tab cấu hình vào trang Hệ Thống:

```php
<?php
use MyPlugin\Modules\Admin\SettingModule;

// Đăng ký tab trong System Settings
add_filter('admin_system_tabs', [SettingModule::class, 'register'], 15);

// Hook lưu cấu hình (tab key = "my_plugin" → hook = admin_system_my_plugin_save)
add_action('admin_system_my_plugin_save', [SettingModule::class, 'save']);
```

---

## 7. Assets — CSS & JS

### 7.1 Cấu trúc thư mục

```
assets/
├── css/
│   ├── style.css           # Frontend
│   └── style.admin.css     # Admin
├── js/
│   ├── script.js           # Frontend
│   └── script.admin.js     # Admin
└── thumb.png               # Ảnh đại diện plugin
```

### 7.2 Helper lấy đường dẫn

```php
$url = asset('my-plugin::css/style.css');
// → https://domain.com/plugins/my-plugin/assets/css/style.css
```

### 7.3 AdminService — Menu & Assets

```php
<?php
namespace MyPlugin\Services;

use SkillDo\Cms\Menu\AdminMenu;
use SkillDo\Cms\Support\Admin;
use SkillDo\Support\Auth;

class AdminService
{
    static function navigation(): void
    {
        // Menu chính
        AdminMenu::add('my-items', 'Quản Lý Items', 'items', [
            'icon'     => '<i class="fa-solid fa-box"></i>',
            'position' => 30
        ]);

        // Submenu
        AdminMenu::addSub('my-items', 'items-list', 'Danh sách', 'items');
        AdminMenu::addSub('my-items', 'items-add', 'Thêm mới', 'items/add');
    }

    static function assets(): void
    {
        Admin::asset()->location('header')
            ->add('my-plugin-admin-css', asset('my-plugin::css/style.admin.css'));

        Admin::asset()->location('footer')
            ->add('my-plugin-admin-js', asset('my-plugin::js/script.admin.js'));
    }
}
```

### 7.4 Frontend Assets

```php
<?php
namespace MyPlugin\Services;

use SkillDo\Cms\Template\Assets\AssetPosition;

class AssetsService
{
    static function web(AssetPosition $header, AssetPosition $footer): void
    {
        $header->add('my-plugin-style', asset('my-plugin::css/style.css'));
        $footer->add('my-plugin-script', asset('my-plugin::js/script.js'));
    }

    static function webVariable(array $variables): array
    {
        $variables['--my-plugin-color'] = config('my-plugin::theme.color', '#333');
        return $variables;
    }
}
```

### AdminMenu API

```php
// Menu chính
AdminMenu::add($key, $title, $slug, [
    'icon'     => '<i class="fa-solid fa-icon"></i>',
    'position' => 30,    // Số nhỏ = hiển thị trên
    'hidden'   => false,
    'count'    => 0,     // Badge count
]);

// Submenu
AdminMenu::addSub($parentKey, $key, $title, $slug, ['position' => 1]);

// Submenu dưới menu có sẵn của CMS
AdminMenu::addSub('marketing', 'my-feature', 'My Feature', 'my-feature');
```

**Position tham khảo:** home=2, page=11, post=21, galleries=31, theme=41, plugins=51, system=61, user=71.

---

## 8. Ajax Handlers

### 8.1 Đăng ký (bootstrap/ajax.php)

```php
<?php
use SkillDo\Cms\Support\Ajax;

Ajax::admin('MyPlugin\Ajax\Admin\ItemAjax::save');    // Chỉ admin
Ajax::client('MyPlugin\Ajax\Web\PublicAjax::search');  // Không cần login
Ajax::login('MyPlugin\Ajax\Web\UserAjax::submit');     // Cần login
```

### 8.2 Ajax Handler Class

```php
<?php
namespace MyPlugin\Ajax\Admin;

use SkillDo\Http\Request;

class ItemAjax
{
    static function save(Request $request): void
    {
        $result = [];
        // ... xử lý logic
        response()->success($result);
    }
}
```

---

## 9. Lifecycle & Hooks hệ thống

### Vòng đời Plugin

1. **Activate** → `active()` trong Main Class → chạy 1 lần khi admin bấm BẬT
2. **Boot** → CMS load autoload, providers, bootstrap files → mỗi request
3. **Uninstall** → `uninstall()` trong Main Class → chạy khi admin bấm GỠ

### Pipeline mỗi request (khi plugin đang BẬT)

1. CMS đọc DB → lấy danh sách plugin đang bật
2. Load PSR-4 autoloader từ `plugin.json`
3. Load `config/` → truy cập qua `config('plugin-id::key')`
4. Load `language/` → truy cập qua `trans('plugin-id::file.key')`
5. Require các files trong `autoload.files`
6. Gọi `register()` và `boot()` của mỗi ServiceProvider
7. Dispatch hook `plugins_loaded` → tất cả plugin đã sẵn sàng

### Hooks hệ thống quan trọng

| Hook | Type | Mô tả |
|------|------|-------|
| `plugins_loaded` | action | Tất cả plugin đã load xong |
| `admin_navigation` | action | Đăng ký menu admin |
| `admin_assets` | action | Nạp CSS/JS cho admin |
| `theme_custom_assets` | action | Nạp CSS/JS cho frontend (nhận $header, $footer) |
| `theme_head_style_variable` | filter | Thêm CSS Variables vào `:root` |
| `admin_system_tabs` | filter | Đăng ký tab System Settings |
| `admin_system_{key}_save` | action | Xử lý lưu System Settings |
| `cms_webhook` | action | Xử lý webhook từ bên ngoài |
| `init` | action | Khởi tạo sau khi CMS boot |

---

## 10. Quy Tắc Đặt Tên

| Thành phần | Quy ước | Ví dụ |
|-----------|---------|-------|
| Thư mục plugin | kebab-case | `my-plugin`, `skd-seo` |
| File main class | trùng tên thư mục hoặc `index.php` | `my-plugin.php` |
| Class name (main) | PascalCase | `MyPlugin` |
| Namespace gốc | PascalCase | `MyPlugin\` |
| ServiceProvider | PascalCase + `ServiceProvider` | `PluginServiceProvider` |
| Controller | PascalCase + `Controller` | `ItemController` |
| Model | PascalCase số ít | `Item` |
| View namespace | `{plugin-folder}::path` | `my-plugin::admin/items/index` |
| Config namespace | `{plugin-folder}::key` | `config('my-plugin::setting.key')` |
| Trans namespace | `{plugin-folder}::file.key` | `trans('my-plugin::messages.title')` |
| Asset path | `{plugin-folder}::path` | `asset('my-plugin::css/style.css')` |

---

## 11. Checklist

### Plugin cơ bản (chỉ hooks, không có CRUD)
1. ✅ Tạo thư mục `plugins/{plugin-id}/`
2. ✅ Tạo `plugin.json` với `name`, `class`, `autoload`
3. ✅ Tạo Main Class (`{plugin-id}.php` hoặc `index.php`)
4. ✅ Tạo `bootstrap/` files đăng ký hooks
5. ✅ (Tùy chọn) Tạo ServiceProvider nếu cần register/boot logic

### Plugin có Admin CRUD
Thêm vào checklist trên:
6. ✅ Tạo Model trong `app/Models/`
7. ✅ Tạo Routes trong `routes/admin.php`
8. ✅ Tạo Controller trong `app/Controllers/Admin/`
9. ✅ Tạo Form + Table (tham khảo skill **module-builder**)
10. ✅ Tạo Views trong `views/admin/`
11. ✅ Đăng ký AdminMenu trong `AdminService`
12. ✅ Đăng ký hooks form trong bootstrap

### Plugin có Frontend
Thêm:
13. ✅ Tạo Routes trong `routes/web.php`
14. ✅ Tạo Controller trong `app/Controllers/Web/`
15. ✅ Tạo Views trong `views/web/`
16. ✅ Đăng ký frontend assets qua hook `theme_custom_assets`

---

## 12. Templates theo cấp độ

### Level 1: Plugin tối giản (chỉ hooks)

Phù hợp cho plugin can thiệp vào hệ thống mà không tạo giao diện riêng.

```
plugins/my-hook-plugin/
├── plugin.json
├── index.php               # Main class rỗng
└── bootstrap/
    └── config.php           # Hooks
```

**Ví dụ thực tế:** plugin `duplicate` — chỉ thêm nút nhân bản vào table sản phẩm/bài viết.

### Level 2: Plugin có Settings + Admin UI

Phù hợp cho plugin cần cấu hình từ admin và giao diện quản lý.

```
plugins/my-settings-plugin/
├── plugin.json
├── my-settings-plugin.php
├── app/
│   ├── Providers/
│   │   └── PluginServiceProvider.php
│   ├── Services/
│   │   └── AdminService.php
│   └── Modules/
│       └── Admin/
│           └── SettingModule.php
├── bootstrap/
│   ├── admin.php
│   └── web.php
├── assets/
│   ├── css/
│   └── js/
└── views/
    └── admin/
```

**Ví dụ thực tế:** plugin `telegram` — cấu hình bot, gửi thông báo.

### Level 3: Plugin đầy đủ CRUD + Frontend

Plugin lớn với model, controller, routes, views.

```
plugins/my-full-plugin/
├── plugin.json
├── index.php
├── app/
│   ├── Ajax/Admin/
│   ├── Controllers/
│   │   ├── Admin/
│   │   └── Web/
│   ├── Models/
│   ├── Modules/Admin/{Module}/
│   │   ├── Form.php
│   │   ├── Table.php
│   │   └── Logger.php
│   ├── Providers/
│   ├── Services/
│   └── Supports/
├── bootstrap/
│   ├── admin.php
│   ├── web.php
│   └── ajax.php
├── routes/
│   ├── admin.php
│   └── web.php
├── config/
│   └── config.php
├── language/
├── views/
│   ├── admin/
│   └── web/
└── assets/
```

**Ví dụ thực tế:** plugin `ProductsFeed`, `sicommerce`.

> **Lưu ý:** Khi plugin cần Admin CRUD (Model, Form, Table, Controller, View), hãy kết hợp với skill **module-builder** để tạo module CRUD hoàn chỉnh.
