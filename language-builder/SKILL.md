---
name: language-builder
description: Hướng dẫn tạo file ngôn ngữ (i18n / Localization) cho Plugin, Theme hoặc Element trong SkillDo CMS v8 - cấu trúc thư mục language/, file mảng PHP, namespace trans(), placeholder, và export JS. Sử dụng skill này khi user yêu cầu tạo ngôn ngữ, dịch thuật, đa ngôn ngữ, i18n, localization, translation cho plugin, theme hoặc element. Bất kỳ khi nào nhắc đến 'tạo ngôn ngữ', 'thêm ngôn ngữ', 'đa ngôn ngữ', 'dịch', 'i18n', 'translation', 'localization', 'trans()', 'file language' cho SkillDo CMS đều nên dùng skill này.
---

# Tạo Ngôn Ngữ (i18n) - SkillDo CMS v8

Hướng dẫn tạo file ngôn ngữ đa ngôn ngữ cho **Plugin**, **Theme** hoặc **Element** trong SkillDo CMS v8.

## Mục lục

1. [Tổng Quan Hệ Thống Ngôn Ngữ](#1-tổng-quan-hệ-thống-ngôn-ngữ)
2. [Ngôn Ngữ Cho Plugin](#2-ngôn-ngữ-cho-plugin)
3. [Ngôn Ngữ Cho Theme](#3-ngôn-ngữ-cho-theme)
4. [Ngôn Ngữ Cho Element](#4-ngôn-ngữ-cho-element)
5. [Cú Pháp File Ngôn Ngữ](#5-cú-pháp-file-ngôn-ngữ)
6. [Sử Dụng Trong JavaScript](#6-sử-dụng-trong-javascript)
7. [Quy Tắc & Best Practices](#7-quy-tắc--best-practices)
8. [Checklist](#8-checklist)

---

## 1. Tổng Quan Hệ Thống Ngôn Ngữ

SkillDo CMS v8 sử dụng hàm `trans()` thống nhất cho toàn bộ hệ thống, phân biệt ngôn ngữ qua **Namespace**.

### Bảng Namespace

| Loại | Cú pháp `trans()` | Thư mục chứa file | Đăng ký |
|------|-------------------|-------------------|---------|
| **Global (Core)** | `trans('file.key')` | `sourcev8/language/{locale}/` | Tự động bởi CMS |
| **Plugin** | `trans('{plugin-id}::file.key')` | `plugins/{plugin-id}/language/{locale}/` | Tự động khi plugin active |
| **Theme** | `trans('theme::file.key')` | `views/{theme-name}/language/{locale}/` | Tự động bởi CMS |
| **Element đơn** | `trans('e-{element-name}::file.key')` | `views/{theme}/elements/{element-name}/language/{locale}/` | Tự động từ widget.json |
| **Element có style** | `trans('e-{element-name}.{style}::file.key')` | `views/{theme}/elements/{element-name}/{style}/language/{locale}/` | Tự động từ widget.json |

### Cơ chế hoạt động

- File ngôn ngữ là **file PHP trả về mảng** (array) chứa các cặp key-value.
- Locale mặc định: `vi` (Tiếng Việt) và `en` (Tiếng Anh).
- Nếu key không tồn tại, `trans()` trả về **đúng chuỗi truyền vào** (dùng để debug).
- CMS tự động load thư mục `language/` nếu đặt đúng vị trí — không cần đăng ký thủ công.

---

## 2. Ngôn Ngữ Cho Plugin

### 2.1 Cấu trúc thư mục

```
plugins/{plugin-id}/
├── language/
│   ├── vi/
│   │   ├── admin.php        # Giao diện quản trị
│   │   ├── messages.php     # Thông báo, alert
│   │   └── {tên-file}.php   # Tùy ý thêm file
│   └── en/
│       ├── admin.php
│       ├── messages.php
│       └── {tên-file}.php
├── plugin.json
└── ...
```

### 2.2 Namespace & cách gọi

Namespace = **tên thư mục plugin** (plugin-id).

```php
// Plugin: skd-seo → Namespace: skd-seo
trans('skd-seo::admin.title');      // Tìm file: plugins/skd-seo/language/{locale}/admin.php → key 'title'
trans('skd-seo::messages.success'); // Tìm file: plugins/skd-seo/language/{locale}/messages.php → key 'success'
```

### 2.3 Đăng ký tự động

CMS Plugin Loader (`LanguageServiceProvider`) tự động quét tất cả plugin đang active và đăng ký namespace dịch thuật. Bạn chỉ cần tạo đúng thư mục `language/` bên trong plugin.

### 2.4 Template file ngôn ngữ cho Plugin

**File `language/vi/admin.php`:**
```php
<?php
return [
    'title'       => 'Tiêu đề quản trị',
    'description' => 'Mô tả plugin',
    'save'        => 'Lưu thiết lập',
    'delete.confirm' => 'Bạn có chắc chắn muốn xóa?',
];
```

**File `language/en/admin.php`:**
```php
<?php
return [
    'title'       => 'Admin Title',
    'description' => 'Plugin Description',
    'save'        => 'Save Settings',
    'delete.confirm' => 'Are you sure you want to delete?',
];
```

### 2.5 Sử dụng trong code Plugin

```php
// Trong Controller, View, Service, Ajax...
echo trans('my-plugin::admin.title');

// Với placeholder
echo trans('my-plugin::messages.welcome', ['name' => 'Hữu Trọng']);
// File: 'welcome' => 'Xin chào :name, chào mừng đến hệ thống.'
// Kết quả: Xin chào Hữu Trọng, chào mừng đến hệ thống.
```

### 2.6 Đăng ký thủ công (nâng cao)

Nếu package nằm ngoài cấu trúc chuẩn plugin, đăng ký trong **ServiceProvider**:

```php
class CustomPackageServiceProvider extends \SkillDo\ServiceProvider
{
    public function boot(): void
    {
        // loadTranslationsFrom(ĐƯỜNG DẪN, NAMESPACE)
        $this->loadTranslationsFrom(__DIR__.'/../language', 'custom-pkg');
    }
}
// Sử dụng: trans('custom-pkg::file.key')
```

---

## 3. Ngôn Ngữ Cho Theme

### 3.1 Cấu trúc thư mục

```
views/{theme-name}/
├── language/
│   ├── vi/
│   │   ├── general.php      # Từ khóa chung (Trang chủ, Xem thêm...)
│   │   ├── auth.php         # Đăng nhập, đăng ký
│   │   ├── contact.php      # Liên hệ
│   │   ├── page.php         # Trang
│   │   ├── post.php         # Bài viết
│   │   └── {tên-file}.php   # Tùy ý
│   └── en/
│       ├── general.php
│       └── ...
├── app/
├── views/
└── ...
```

### 3.2 Namespace & cách gọi

Namespace cố định: **`theme`**

```php
// Luôn bắt đầu bằng 'theme::'
trans('theme::general.home');        // → 'Trang chủ'
trans('theme::general.view.more');   // → 'Xem thêm'
trans('theme::auth.login');          // → 'Đăng nhập'
trans('theme::contact.form.title');  // → 'Liên hệ với chúng tôi'
```

### 3.3 Template file ngôn ngữ cho Theme

**File `language/vi/general.php`:**
```php
<?php
return [
    'home'      => 'Trang chủ',
    'view'      => 'Xem',
    'view.more' => 'Xem thêm',
    'send'      => 'Gửi',
    'search'    => 'Tìm kiếm',
    'confirm'   => 'Xác nhận',
    'update'    => 'Cập nhật',
    'product'   => 'Sản phẩm',
    'share'     => 'Chia sẻ',
    'contact'   => 'Liên hệ',
];
```

**File `language/en/general.php`:**
```php
<?php
return [
    'home'      => 'Home',
    'view'      => 'View',
    'view.more' => 'View More',
    'send'      => 'Send',
    'search'    => 'Search',
    'confirm'   => 'Confirm',
    'update'    => 'Update',
    'product'   => 'Product',
    'share'     => 'Share',
    'contact'   => 'Contact',
];
```

### 3.4 Sử dụng trong Blade Template

```blade
<a href="#" class="btn btn-primary">{{ trans('theme::general.view.more') }}</a>

<p>{{ trans('theme::page.welcome_message', ['store' => 'Sikido']) }}</p>

{{-- Fallback: nếu key không tồn tại, in nguyên chuỗi --}}
<button>{{ trans('theme::general.non_exist_key') }}</button>
```

### 3.5 Ghi đè ngôn ngữ Plugin từ Theme

Theme có quyền ghi đè ngôn ngữ Plugin. Tạo file tại:
```
views/{theme-name}/language/{locale}/{plugin-id}/{file}.php
```
Ví dụ: Ghi đè `trans('skd-seo::admin.title')` từ Theme:
```
views/theme-store/language/vi/skd-seo/admin.php
```

---

## 4. Ngôn Ngữ Cho Element

Element (widget Page Builder) có 2 cách sử dụng ngôn ngữ:
- **Cách 1**: Dùng namespace chung của Theme (`theme::`) hoặc Plugin (`plugin-id::`)
- **Cách 2**: Dùng **namespace riêng cho Element** (prefix `e-`)

### 4.1 Khi nào dùng namespace riêng cho Element?

Dùng namespace riêng (`e-`) khi Element phức tạp, có nhiều label cần dịch và muốn tách biệt file language riêng cho từng Element thay vì viết tập trung vào file theme.

### 4.2 Cấu trúc thư mục — Element đơn (không có style variant)

```
views/{theme-name}/elements/{element-name}/
├── {element-name}.widget.php
├── views/
│   └── view.blade.php
├── language/                    ← Thư mục language cho element
│   ├── vi/
│   │   └── main.php             ← File language (tên tùy ý)
│   └── en/
│       └── main.php
└── assets/
```

**Namespace**: `e-{element-name}`

```php
// Element: search-bar → Namespace: e-search-bar
trans('e-search-bar::main.search');    // → 'Kết quả tìm kiếm'
```

### 4.3 Cấu trúc thư mục — Element có style variant

```
views/{theme-name}/elements/{element-name}/
├── style1/
│   ├── {element-name}-style1.widget.php
│   ├── views/
│   │   └── view.blade.php
│   ├── language/                ← Language cho style1
│   │   ├── vi/
│   │   │   └── main.php
│   │   └── en/
│   │       └── main.php
│   └── assets/
├── style2/
│   ├── language/                ← Language cho style2
│   │   └── ...
│   └── ...
```

**Namespace**: `e-{element-name}.{style}`

```php
// Element: auth-button, style: style1 → Namespace: e-auth-button.style1
trans('e-auth-button.style1::main.account');   // → 'Tài khoản'
trans('e-auth-button.style1::main.logout');     // → 'Đăng xuất'
```

### 4.4 Cách CMS tạo Element Namespace

CMS đọc `path` từ widget.json, bỏ file cuối, thay `/` thành `.`, bỏ prefix `widget.elements.`, thêm prefix `e-`:

```
path: "widget/elements/auth-button/style1/auth-button-style1.widget.php"
     → bỏ file: "widget/elements/auth-button/style1"
     → thành dot: "widget.elements.auth-button.style1"
     → bỏ prefix: "auth-button.style1"
     → thêm e-:  "e-auth-button.style1"
```

### 4.5 Template file ngôn ngữ cho Element

**File `language/vi/main.php`:**
```php
<?php
return [
    'account'   => 'Tài khoản',
    'info'      => 'Thông tin tài khoản',
    'order'     => 'Danh sách đơn hàng',
    'logout'    => 'Đăng xuất',
];
```

**File `language/en/main.php`:**
```php
<?php
return [
    'account'   => 'Account',
    'info'      => 'Account Information',
    'order'     => 'Order List',
    'logout'    => 'Logout',
];
```

### 4.6 Sử dụng trong class Element

```php
class AuthButtonStyle1 extends Element
{
    public function name()
    {
        // Dùng namespace element riêng
        return trans('e-auth-button.style1::main.account');
    }

    public function form(): void
    {
        $this->tabs('generate')->adds(function (\SkillDo\Cms\Form\Form $form) {
            $form->text('title', [
                'label' => trans('e-auth-button.style1::main.info'),
            ]);
        });
        parent::form();
    }

    public function widget(): void
    {
        // Trong view blade
        // {{ trans('e-auth-button.style1::main.logout') }}
    }
}
```

### 4.7 Sử dụng ngôn ngữ Theme hoặc Core trong Element

Không bắt buộc phải dùng namespace riêng `e-`. Element có thể gọi ngôn ngữ từ bất kỳ nguồn nào:

```php
// Dùng ngôn ngữ Core (các từ cơ bản đã có sẵn)
trans('field.color');         // Màu sắc
trans('field.font_size');     // Cỡ chữ
trans('button.save');         // Lưu

// Dùng ngôn ngữ Theme
trans('theme::general.view.more');   // Xem thêm

// Dùng ngôn ngữ Plugin
trans('skd-seo::admin.keywords');    // Từ khóa SEO
```

Lưu ý: Framework đã xây dựng sẵn gói language lớn tại `sourcev8/language/{locale}/` cho các từ khóa thiết kế cơ bản (margin, padding, color, background...). Hãy kiểm tra file `field.php`, `button.php`, `general.php` trong thư mục `language/` trước khi tự tạo key trùng.

---

## 5. Cú Pháp File Ngôn Ngữ

### 5.1 Cấu trúc file cơ bản

Mỗi file trả về một mảng PHP với các cặp `'key' => 'value'`:

```php
<?php
return [
    'key_simple'    => 'Giá trị đơn giản',
    'key.nested'    => 'Dùng dấu chấm thay vì mảng lồng',
    'key_with_html' => '<strong>Có thể chứa HTML</strong>',
];
```

### 5.2 Dấu chấm (dot notation) trong key

SkillDo CMS **hỗ trợ cả 2 cách** viết key cho phân cấp:

**Cách 1: Dấu chấm trong key (flat) — KHUYẾN NGHỊ**
```php
<?php
return [
    'ajax.add.success'  => 'Thêm thành công',
    'ajax.add.error'    => 'Thêm thất bại',
    'cart.empty.title'  => 'Giỏ hàng trống',
];
```

**Cách 2: Mảng lồng (nested)**
```php
<?php
return [
    'banner' => [
        'title'         => 'Khối Banner',
        'heading_label' => 'Tiêu đề lớn',
    ],
];
```

Gọi bằng `trans()` như nhau:
```php
trans('plugin::file.ajax.add.success');
trans('theme::element.banner.title');
```

### 5.3 Placeholder (tham số động)

Dùng `:tên_biến` trong chuỗi gốc, truyền mảng ở tham số thứ 2:

```php
// File language:
return [
    'welcome'   => 'Xin chào :name, mừng bạn đến với :system.',
    'cart.added' => 'Sản phẩm <strong>:productName</strong> đã được thêm vào giỏ hàng',
];

// Sử dụng:
trans('my-plugin::messages.welcome', ['name' => 'Hữu Trọng', 'system' => 'CMS v8']);
// → Xin chào Hữu Trọng, mừng bạn đến với CMS v8.

trans('sicommerce::cart.ajax.add.success', ['productName' => 'iPhone 16']);
// → Sản phẩm <strong>iPhone 16</strong> đã được thêm vào giỏ hàng
```

---

## 6. Sử Dụng Trong JavaScript

File JS không thể gọi `trans()` PHP. SkillDo cung cấp cách export ngôn ngữ sang JS.

### 6.1 Export từ PHP sang JS (trong Blade head)

```blade
{!! Skd::head() !!}

<script>
    window.Skd_Langs = {
       "button": @json(trans('theme::button')),
       "alert": @json(trans('theme::alert'))
    };
</script>
```

### 6.2 Sử dụng trong JS

```javascript
// Cách 1: Thư viện Translator tích hợp sẵn
alert(Skd.lang.get('alert.success_add_cart'));

// Cách 2: Có kèm tham số
let msg = Skd.lang.get('alert.error_auth', { name: "Nguyễn Văn A" });
```

---

## 7. Quy Tắc & Best Practices

### Quy tắc bắt buộc

| Quy tắc | Mô tả |
|---------|-------|
| File trả về mảng | Mỗi file `.php` phải bắt đầu bằng `<?php` và `return [...]` |
| Đặt đúng thư mục | `language/{locale}/{file}.php` — locale phải là `vi`, `en`... |
| Namespace đúng | Plugin dùng `{plugin-id}::`, Theme dùng `theme::`, Element dùng `e-{namespace}::` |
| Key duy nhất | Không trùng key trong cùng 1 file |
| Đồng bộ key giữa các locale | File `vi/admin.php` và `en/admin.php` phải có **cùng danh sách key** |

### Best Practices

1. **Kiểm tra Core trước khi tạo key mới**: Các từ cơ bản (`color`, `save`, `delete`, `margin`...) đã có sẵn trong `language/{locale}/field.php`, `button.php`, `general.php`. Dùng `trans('field.color')` thay vì tự tạo.

2. **Tổ chức file theo ngữ cảnh**: Chia nhỏ file theo chức năng (`admin.php`, `cart.php`, `checkout.php`) thay vì gom hết vào 1 file lớn.

3. **Dùng dot notation cho key**: Dùng `'cart.empty.title'` thay vì mảng lồng `'cart' => ['empty' => ['title' => ...]]` để dễ đọc và tra cứu.

4. **Luôn tạo ít nhất 2 locale**: `vi/` và `en/` là tối thiểu.

5. **Không hardcode chuỗi tĩnh**: Thay `'label' => 'Tiêu đề'` bằng `'label' => trans('my-plugin::form.title')` trong Controller, Form, Element.

6. **File Element nên đặt tên `main.php`**: Convention cho Element language — dùng `main.php` làm file chính.

---

## 8. Checklist

### Checklist tạo ngôn ngữ Plugin
1. ✅ Tạo thư mục `plugins/{plugin-id}/language/vi/`
2. ✅ Tạo thư mục `plugins/{plugin-id}/language/en/`
3. ✅ Tạo các file `.php` trả về mảng (`admin.php`, `messages.php`...)
4. ✅ Đồng bộ key giữa `vi/` và `en/`
5. ✅ Thay thế chuỗi hardcode trong code bằng `trans('{plugin-id}::file.key')`
6. ✅ Kiểm tra fallback — `trans()` trả về chuỗi gốc nếu key không tồn tại

### Checklist tạo ngôn ngữ Theme
1. ✅ Tạo thư mục `views/{theme-name}/language/vi/`
2. ✅ Tạo thư mục `views/{theme-name}/language/en/`
3. ✅ Tạo các file `.php` trả về mảng
4. ✅ Đồng bộ key giữa `vi/` và `en/`
5. ✅ Sử dụng `trans('theme::file.key')` trong Blade views
6. ✅ (Tùy chọn) Export JS nếu cần dùng trong JavaScript

### Checklist tạo ngôn ngữ Element
1. ✅ Xác định loại Element: đơn hay có style variant
2. ✅ Tạo thư mục `language/vi/` bên trong thư mục widget element (cùng cấp với `views/`)
3. ✅ Tạo file `main.php` (hoặc tên tùy ý) trả về mảng
4. ✅ Xác định đúng namespace: `e-{element-name}` hoặc `e-{element-name}.{style}`
5. ✅ Đồng bộ key giữa `vi/` và `en/`
6. ✅ Sử dụng `trans('e-{namespace}::file.key')` trong class Element và Blade view
7. ✅ Kiểm tra xem key đã có trong Core (`field.php`, `button.php`) chưa trước khi tạo mới
