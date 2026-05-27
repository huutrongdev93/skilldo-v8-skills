---
name: element-builder
description: Hướng dẫn tạo Element (widget) cho Page Builder trong SkillDo CMS v8 - cấu trúc class, form, view, assets, cssBuilder và đăng ký trong widget.json.
---

# Hướng Dẫn Tạo Element Trong Theme Store

> Tài liệu này hướng dẫn AI tạo các element trong `views/theme-store/widget/elements/`.

## 1. Tổng Quan

Element là các widget nhỏ (heading, image, button, icon-box...) được sử dụng trong hệ thống page builder của SkillDo CMS. Mỗi element kế thừa từ `SkillDo\Cms\Element\Element` và phải implement interface `ElementInterface`.

### Phân biệt 2 loại cấu trúc thư mục

**Loại 1: Element đơn (không có style variant)**
```
elements/
└── {element-name}/
    ├── {element-name}.widget.php    # File PHP chính
    ├── views/
    │   └── view.blade.php           # Template hiển thị
    └── assets/                      # (tùy chọn)
        ├── {name}.less
        ├── {name}.css
        └── {name}.js
```
Ví dụ: `Image`, `icon-box`, `divider`, `text-editor`, `counter`, `slider`

**Loại 2: Element có nhiều style variant**
```
elements/
└── {element-name}/
    ├── style1/
    │   ├── {element-name}-style1.widget.php
    │   ├── views/
    │   │   └── view.blade.php
    │   └── assets/
    │       ├── {name}.less
    │       └── {name}.js
    ├── style2/
    │   └── ...
    └── styleN/
        └── ...
```
Ví dụ: `heading`, `button`, `posts`, `products`, `feedback`, `items`

## 2. Đăng Ký Element Trong widget.json

File `views/theme-store/widget/widget.json` chứa cấu hình đăng ký tất cả elements. Element được đăng ký trong key `"elements"` với các nhóm:

| Nhóm | Mô tả |
|------|--------|
| `header` | Element dùng trong header (logo, cart, search...) |
| `footer` | Element dùng trong footer |
| `general` | Element chung cho mọi vị trí |
| `post` | Element cho trang chi tiết bài viết |
| `page` | Element cho trang chi tiết page |
| `products_category` | Element cho trang danh mục sản phẩm |

### Cấu trúc đăng ký

```json
{
    "elements": {
        "general": {
            "TenClassElement": {
                "path": "widget/elements/{element-name}/{file}.widget.php"
            }
        }
    }
}
```

**Element có ajax** (dùng cho products):
```json
{
    "ProductsElementStyle1": {
        "path": "widget/elements/products/style1/products-style1.widget.php",
        "ajax": {
            "client": "ProductsElementStyle1::loadProduct"
        }
    }
}
```

## 3. Cấu Trúc Class Element

### 3.1 Skeleton cơ bản

```php
<?php

use SkillDo\Cms\Element\Element;
use SkillDo\Cms\Support\Theme;

class MyElement extends Element
{
    public function __construct()
    {
        // Param 1: key (trùng tên class), Param 2: tên hiển thị
        parent::__construct('MyElement', 'Tên hiển thị');

        // Đăng ký assets (tùy chọn)
        $this->assets('assets/my-element.less');
        $this->assets('assets/my-element.js');

        // Thêm CSS class cho wrapper (tùy chọn)
        $this->configClass('custom-class');

        // Thêm tags (tùy chọn)
        $this->setTags('tag-name');
    }

    // REQUIRED: Icon hiển thị trong sidebar builder
    public function icon(): string
    {
        return 'icon-name'; // hoặc '<i class="fa-duotone fa-solid fa-icon"></i>'
    }

    // REQUIRED: Nhóm category trong sidebar
    public function category(): string
    {
        return 'basic'; // basic | heading | general | post | ecommerce
    }

    // REQUIRED: Khai báo form cấu hình
    public function form(): void
    {
        // Tab "Nội dung"
        $this->tabs('generate')->adds(function (\SkillDo\Cms\Form\Form $form)
        {
            // Các form field ở đây
        });

        // Tab "Kiểu dáng" (tùy chọn)
        $this->tabs('style')->adds(function (\SkillDo\Cms\Form\Form $form)
        {
            // Các form field style ở đây
        });

        // Tab "Nâng cao" (tùy chọn - đã có sẵn spacing + scroll effects)
        $this->tabs('advanced')->adds(function (\SkillDo\Cms\Form\Form $form)
        {
            // Thêm field vào tab advanced
        });

        // GỌI parent::form() ở cuối
        parent::form();
    }

    // REQUIRED: Render HTML
    public function widget(): void
    {
        Theme::view($this->getDir().'views/view', [
            'options' => $this->options,
        ]);
    }

    // Giá trị mặc định cho options
    public function default(): void
    {
        $defaults = [
            'field1' => 'value1',
            'field2' => 'value2',
        ];

        foreach ($defaults as $key => $value)
        {
            $this->options->{$key} = $this->options->{$key} ?? $value;
        }
    }

    // Build CSS động (tùy chọn)
    public function cssBuilder(): string
    {
        $this->cssSelector('.my-selector', [
            'data'  => $this->options->myStyle ?? [],
            'style' => 'text',
        ]);

        return $this->cssBuild();
    }
}
```

### 3.2 Các method quan trọng từ class cha

| Method | Mô tả |
|--------|--------|
| `$this->getDir()` | Trả về đường dẫn thư mục chứa file widget hiện tại |
| `$this->assets($path)` | Đăng ký file CSS/LESS/JS (đường dẫn tương đối từ thư mục widget) |
| `$this->configClass($class)` | Thêm CSS class vào wrapper |
| `$this->setTags(...$tags)` | Gắn tags cho element |
| `$this->options` | Object chứa tất cả giá trị form đã lưu |
| `$this->id` | ID duy nhất của widget instance |
| `$this->name` | Tên widget |
| `$this->key` | Key duy nhất (tên class) |
| `$this->empty()` | Trả về HTML placeholder khi element rỗng |
| `$this->groupFormBox($text, $id, $active)` | Tạo collapsible group trong tab style |
| `$this->tabs($key)` | Truy cập tab (`generate`, `style`, `advanced`) |
| `$this->widgetContainer($widgets)` | Render danh sách nested widgets (dùng cho tabs) |

### 3.3 CSS Builder Methods

| Method | Mô tả |
|--------|--------|
| `$this->cssSelector($selector, ...$properties)` | Áp dụng style cho selector |
| `$this->cssVariables($key, $value)` | Set CSS custom property |
| `$this->cssStyle($path, $args)` | Set inline style |
| `$this->cssBuild()` | Build và trả về CSS string (luôn gọi cuối cùng) |
| `$this->resetCssBuild()` | Reset CSS builder |

**Các style type cho cssSelector:**

```php
// Text style (font-size, color, font-weight...)
$this->cssSelector('.title', [
    'data'  => $this->options->titleStyle ?? [],
    'style' => 'text',
]);

// Box style (background, border, padding, shadow...)
$this->cssSelector('.box', [
    'data'  => $this->options->boxStyle ?? [],
    'style' => 'box',
]);

// Border style
$this->cssSelector('.border', [
    'data'  => $this->options->borderStyle ?? [],
    'style' => 'border',
]);

// Background style
$this->cssSelector('.bg', [
    'data'  => $this->options->bgStyle ?? [],
    'style' => 'background',
]);

// Background color
$this->cssSelector('.bg', [
    'data'  => $this->options->bgColor ?? [],
    'style' => 'backgroundColor',
]);

// Button style
$this->cssSelector('.btn', [
    'data'  => $this->options->btnStyle ?? [],
    'style' => 'button',
]);

// Box shadow
$this->cssSelector('.shadow', [
    'data'  => $this->options->shadow ?? [],
    'style' => 'boxShadow',
]);

// Spacing (margin, padding)
$this->cssSelector('.space', [
    'data'  => $this->options->spacing ?? [],
    'style' => 'spacing',
]);

// Hover states - dùng array selector
$this->cssSelector([
    'normal' => '.item .title',
    'hover'  => '.item:hover .title',
], [
    'data'  => $this->options->titleStyle ?? [],
    'style' => 'text',
]);

// Nhiều property cho 1 selector
$this->cssSelector('.tab-button',
    ['data' => $this->options->tabBg ?? [], 'style' => 'background'],
    ['data' => $this->options->tabBorder ?? [], 'style' => 'border'],
    ['data' => $this->options->tabShadow ?? [], 'style' => 'boxShadow']
);
```

## 4. Form Fields Thường Dùng

### 4.1 Tab Generate (Nội dung)

```php
$this->tabs('generate')->adds(function (\SkillDo\Cms\Form\Form $form)
{
    // === INPUT CƠ BẢN ===
    $form->text('fieldName', ['label' => 'Label', 'language' => true]);
    $form->textarea('description', ['label' => 'Mô tả', 'language' => true]);
    $form->number('count', ['label' => 'Số lượng', 'value' => 10]);
    $form->wysiwyg('content', ['label' => 'Nội dung', 'language' => true]);

    // === MEDIA ===
    $form->image('img', ['label' => 'Hình ảnh']);
    $form->fontIcon('icon', ['label' => 'Icon']);

    // === SELECT ===
    $form->select('field', ['label' => 'Label'])->options(['key' => 'Value']);
    $form->select2('field', ['label' => 'Label'])->options(['key' => 'Value']);

    // === TAB (radio buttons dạng tab) ===
    $form->tab('align', ['label' => 'Căn chỉnh'])->options([
        'start'  => '<i class="fa-thin fa-align-left"></i>&nbsp;Trái',
        'center' => '<i class="fa-thin fa-align-center"></i>&nbsp;Giữa',
        'end'    => 'Phải&nbsp;<i class="fa-thin fa-align-right"></i>',
    ]);

    // === SWITCH ===
    $form->switch('enabled', ['label' => 'Bật/Tắt'])->display('inline');

    // === RESPONSIVE INPUT (desktop/tablet/mobile) ===
    $form->addResponsive('numberShow', [
        'label' => 'Số hiển thị trên 1 hàng',
        'type'  => \SkillDo\Cms\Form\Field\NumericSelector::class,
        'min'   => 1,
        'max'   => 6,
    ]);
    // Tạo ra: $this->options->desktopNumberShow, tabletNumberShow, mobileNumberShow

    $form->addResponsive('width', [
        'label' => 'Chiều rộng',
        'type'  => \SkillDo\Cms\Form\Field\Range::class,
        'min'   => 10,
        'max'   => 100,
        'step'  => 1,
    ]);

    $form->inputResponsive('iconSize', [
        'label' => 'Kích thước icon',
        'input' => 'number',
    ]);
    // Tạo ra: $this->options->iconSize['desktop'], ['tablet'], ['mobile']

    // === REPEATER (danh sách items) ===
    $form->repeater('items', [
        'label'  => 'Danh sách',
        'fields' => function (\SkillDo\Cms\Form\Form $form)
        {
            $form->image('image', ['label' => 'Ảnh']);
            $form->text('title', ['label' => 'Tiêu đề', 'language' => true]);
            $form->textarea('content', ['label' => 'Nội dung', 'language' => true]);
        }
    ]);

    // === DATA SOURCE ===
    $form->postCategory('cateId', ['label' => 'Danh mục bài viết']);
    $form->productsCategories('categoryId', ['label' => 'Danh mục sản phẩm']);

    // === CONDITIONAL DISPLAY ===
    $form->number('time', ['label' => 'Thời gian'])
        ->condition('display[type]', [0]); // Chỉ hiện khi display[type] = 0

    // === LAYOUT ===
    // start: số cột (tổng 12)
    $form->number('width', ['label' => 'Rộng', 'start' => 6]);
    $form->number('height', ['label' => 'Cao', 'start' => 6]);
});
```

### 4.2 Tab Style (Kiểu dáng)

```php
$this->tabs('style')->adds(function (\SkillDo\Cms\Form\Form $form)
{
    // === BUILDING FIELDS (Style phức hợp) ===

    // Text building: font, size, color, weight, align, line-height...
    $form->textBuilding('titleStyle', [
        'customInput' => [
            'colorHover'  => true,  // Thêm tab hover color
            'tabAdvanced' => false, // Ẩn tab advanced
            'tabHover'    => true,  // Thêm tab hover
            'tabActive'   => true,  // Thêm tab active
        ],
    ])->popup(false);

    // Box building: background, border, padding, border-radius, shadow...
    $form->boxBuilding('boxStyle', [
        'customInput' => ['colorHover' => true],
    ])->popup(false);

    // Button building: style cho button
    $form->buttonBuilding('btnStyle')->popup(false);

    // Color building
    $form->colorBuilding('bgColor', ['label' => 'Màu nền'])->display('inline');

    // === STYLE FIELDS ĐƠN LẺ ===
    $form->color('iconColor', ['label' => 'Màu icon']);
    $form->background('bgStyle', ['label' => 'Nền'])->display('inline');
    $form->border('borderStyle', ['label' => 'Viền']);
    $form->boxShadow('shadowStyle', ['label' => 'Đổ bóng'])->display('inline');
    $form->spacing('spacingStyle')->customInput(['margin' => false]);
    $form->range('size', ['label' => 'Kích thước', 'min' => 10, 'max' => 50]);
    $form->number('width', ['label' => 'Chiều rộng']);

    // === COLLAPSIBLE GROUPS ===
    $form->addGroup(function (\SkillDo\Cms\Form\Form $form)
    {
        $form->boxBuilding('boxStyle')->popup(false);
    }, $this->groupFormBox('Tên group', 'groupId', true)); // true = mở mặc định

    // === TABBED STATES (Normal/Hover/Active) ===
    $form->addTab(function (\SkillDo\Cms\Form\Form $form) {
        $form->background('bg', ['label' => 'Nền'])->display('inline');
    }, 'tab_normal', ['label' => 'Bình thường', 'active' => true]);

    $form->addTab(function (\SkillDo\Cms\Form\Form $form) {
        $form->background('bgHover', ['label' => 'Nền'])->display('inline');
    }, 'tab_hover', ['label' => 'Di chuột']);
});
```

## 5. View Template (Blade)

File: `views/view.blade.php`

```blade
{{-- Ví dụ đơn giản --}}
<div class="my-element">
    @if(!empty($options->title))
    <{!! $options->titleTag !!} class="title">{!! $options->title !!}</{!! $options->titleTag !!}>
    @endif
    @if(!empty($options->description))
    <div class="description">{!! $options->description !!}</div>
    @endif
</div>
```

**Quy tắc:**
- Biến truyền vào view qua mảng param thứ 2 của `Theme::view()`
- Dùng `{!! !!}` cho HTML content, `{{ }}` cho text thuần
- Dùng `@if(!empty(...))` để kiểm tra trước khi render
- Dùng `Image::{size}($path, $alt)->html()` để render ảnh

## 6. Assets (LESS/CSS/JS)

### LESS file

```less
// Selector gốc = tên class element (key)
.MyElement {
    .my-selector {
        font-size: 14px;
        color: var(--my-custom-var, #333);
    }
}

// Responsive
@media (max-width: 1024px) {
    .MyElement {
        .my-selector {
            font-size: var(--my-var-tablet, 14px);
        }
    }
}

@media (max-width: 768px) {
    .MyElement {
        .my-selector {
            font-size: var(--my-var-mobile, 14px);
        }
    }
}
```

**Quan trọng:** Selector gốc trong LESS phải trùng với `key` (tên class) của element.

### JS file

File JS dùng pattern **ES6 class** + hook đăng ký qua `elementorFrontend`. Mỗi file JS phải tuân theo cấu trúc sau:

#### Cấu trúc bắt buộc

```javascript
class MyWidgetElement
{
    /**
     * @param {jQuery} scope - jQuery wrapper element của widget (tự động truyền vào)
     * @param {jQuery} $ - jQuery reference
     */
    constructor(scope, $)
    {
        this.scope = scope;
        this.$ = $;

        // Tìm container chính trong scope
        this.container = scope.find('.my-container');

        if (!this.container.length) return;

        // Khởi tạo logic
        this.init();
    }

    init()
    {
        // Logic khởi tạo: bindEvents, observer, slider...
    }

    /**
     * REQUIRED: Dọn dẹp khi widget bị xóa khỏi page builder
     * - Hủy event listeners
     * - Hủy observers, timers, animation frames
     * - Hủy Swiper/GSAP instances
     */
    destroy()
    {
        this.scope = null;
    }
}

// === HOOK ĐĂNG KÝ (BẮT BUỘC) ===
$(window).on('elementor/frontend/init', function ()
{
    elementorFrontend.hooks.addAction(
        'frontend/ready/{TenClass}.default',   // {TenClass} = tên class PHP element
        function (scope, $)
        {
            const instance = new MyWidgetElement(scope, $);

            scope.data('onDestroy', function () {
                instance.destroy();
            });
        }
    );
});
```

#### Quy tắc hook đăng ký

| Thành phần | Giá trị | Ví dụ |
|------------|---------|-------|
| Hook action | `'frontend/ready/{TenClassPHP}.default'` | `'frontend/ready/CounterElement.default'` |
| `{TenClassPHP}` | Phải **trùng chính xác** tên class PHP element | `FaqElement`, `ProductsElementStyle1` |
| `scope` | jQuery wrapper element (tự động truyền) | - |
| `onDestroy` | Callback gọi khi widget bị xóa/rebuild trong builder | - |

#### Biến global có sẵn

| Biến | Mô tả |
|------|--------|
| `$` | jQuery |
| `ajax` | URL endpoint cho Ajax request |
| `request` | HTTP client (`request.post(ajax, data)`) |
| `Swiper` | Swiper library (nếu đã load) |
| `gsap` | GSAP animation library (nếu đã load) |
| `SkilldoMessage` | Hiển thị toast message (`SkilldoMessage.error(msg)`) |
| `elementorFrontend` | Page builder frontend API |

#### Pattern 1: DOM Interaction đơn giản

Dùng cho element cần bind events click, toggle, keyboard...

```javascript
class FaqElement
{
    constructor(scope, $)
    {
        this.scope = scope;
        this.container = scope.find('.faq-container');
        if (!this.container.length) return;
        this.bindEvents();
    }

    bindEvents()
    {
        const self = this;
        this.scope.find('.faq-question').on('click', function(e) {
            e.preventDefault();
            const $item = $(this).closest('.faq-item');
            self.toggleItem($item);
        });
    }

    toggleItem($item) { /* ... */ }

    destroy()
    {
        this.scope.find('.faq-question').off('click');
    }
}
```

#### Pattern 2: Swiper Slider

Dùng cho element có carousel/slider. Luôn hủy Swiper trong `destroy()`.

```javascript
class MySliderElement
{
    constructor(scope, $)
    {
        this.scope = scope;
        this.$ = $;
        this.container = scope.find('.js_my_data');
        if (!this.container.length) return;

        // Parse options từ data attribute
        try {
            this.options = this.container.data('options');
            if (typeof this.options === 'string') {
                this.options = JSON.parse(this.options);
            }
        } catch (e) {
            console.error('Error parsing options', e);
            return;
        }

        this.swiperInstance = null;
        this.slider();
    }

    shouldBeEnabled($carousel, numberShow)
    {
        const slidesCount = $carousel.find('.swiper-slide').length;
        return { loop: slidesCount >= numberShow };
    }

    slider()
    {
        const swiperContainer = this.scope.find('.swiper');
        const btnNext = this.scope.find('.next');
        const btnPrev = this.scope.find('.prev');

        let gutter = parseInt(getComputedStyle(document.body)
            .getPropertyValue('--bs-gutter-x')) || 30;

        let config = {
            ...this.shouldBeEnabled(swiperContainer, parseInt(this.options.desktopNumberShow)),
            autoplay: {
                delay: parseInt(this.options.display.time) * 1000,
                disableOnInteraction: false
            },
            speed: 500,
            slidesPerView: parseInt(this.options.desktopNumberShow),
            spaceBetween: gutter,
            breakpoints: {
                0: {
                    ...this.shouldBeEnabled(swiperContainer, parseInt(this.options.mobileNumberShow)),
                    slidesPerView: parseInt(this.options.mobileNumberShow)
                },
                768: {
                    ...this.shouldBeEnabled(swiperContainer, parseInt(this.options.tabletNumberShow)),
                    slidesPerView: parseInt(this.options.tabletNumberShow)
                },
                1000: {
                    ...this.shouldBeEnabled(swiperContainer, parseInt(this.options.desktopNumberShow)),
                    slidesPerView: parseInt(this.options.desktopNumberShow)
                },
            },
        };

        if (swiperContainer.length) {
            this.swiperInstance = new Swiper(swiperContainer[0], config);
        }

        btnNext.on('click', () => {
            if (this.swiperInstance) this.swiperInstance.slideNext();
        });
        btnPrev.on('click', () => {
            if (this.swiperInstance) this.swiperInstance.slidePrev();
        });
    }

    destroy()
    {
        if (this.swiperInstance) {
            this.swiperInstance.destroy(true, true);
            this.swiperInstance = null;
        }
        this.scope.find('.next').off('click');
        this.scope.find('.prev').off('click');
    }
}
```

#### Pattern 3: Ajax + IntersectionObserver (Lazy Load)

Dùng cho element tải dữ liệu từ server khi vào viewport (products, posts...).

```javascript
class MyProductElement
{
    constructor(scope, $)
    {
        this.scope = scope;
        this.$ = $;
        this.container = scope.find('.js_my_data');
        if (!this.container.length) return;

        this.options = this.container.data('options');
        this.observer = null;
        this.isLoaded = false;

        this.initIntersectionObserver();
    }

    initIntersectionObserver()
    {
        if ('IntersectionObserver' in window)
        {
            this.observer = new IntersectionObserver((entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting && !this.isLoaded) {
                        this.isLoaded = true;
                        this.loadData();
                        this.observer.disconnect();
                    }
                });
            }, { root: null, rootMargin: '0px 0px 200px 0px', threshold: 0 });

            this.observer.observe(this.container[0]);
        }
        else
        {
            this.loadData(); // Fallback cho trình duyệt cũ
        }
    }

    loadData()
    {
        let data = {
            action: 'MyProductElement::loadProduct',  // Khớp với ajax trong widget.json
            options: this.options,
        };

        request.post(ajax, data).then((response) => {
            if (response.status === 'success') {
                // Render response.data.items vào DOM
            }
        }).catch(err => {
            console.error('Ajax Error:', err);
        });
    }

    destroy()
    {
        if (this.observer) {
            this.observer.disconnect();
            this.observer = null;
        }
    }
}
```

#### Pattern 4: Animation với IntersectionObserver

Dùng cho element cần animate khi vào viewport (counter, progress bar...).

```javascript
class CounterWidget
{
    constructor(scope, $)
    {
        this.scope = scope;
        this.counter = scope.find('.counter-card');
        this.duration = this.counter.data('duration');
        this.observer = null;
        this.animationFrames = [];

        if (this.counter.length) this.init();
    }

    init()
    {
        this.observer = new IntersectionObserver((entries) => {
            if (entries[0].isIntersecting) {
                this.animateCounter(entries[0].target);
                this.observer.disconnect();
            }
        }, { root: null, threshold: 0.2 });

        this.observer.observe(this.counter[0]);
    }

    animateCounter(el)
    {
        const targetEl = el.querySelector('.counter-target');
        const target = +targetEl.getAttribute('data-target');
        const startTime = performance.now();

        const updateCount = (currentTime) => {
            const progress = Math.min((currentTime - startTime) / this.duration, 1);
            targetEl.innerText = Math.floor(progress * target).toLocaleString('en-US');
            if (progress < 1) {
                this.animationFrames.push(requestAnimationFrame(updateCount));
            }
        };

        this.animationFrames.push(requestAnimationFrame(updateCount));
    }

    destroy()
    {
        if (this.observer) {
            this.observer.disconnect();
            this.observer = null;
        }
        this.animationFrames.forEach(id => cancelAnimationFrame(id));
        this.animationFrames = [];
    }
}
```

#### Quy tắc chung cho JS file

1. **Luôn dùng ES6 class** — không dùng function constructor truyền thống
2. **Tên class JS tự do** — không bắt buộc trùng tên class PHP, nhưng khuyến khích đặt giống hoặc tương tự
3. **Hook action phải trùng tên class PHP** — `'frontend/ready/{TenClassPHP}.default'`
4. **Luôn implement `destroy()`** — dọn dẹp events, observers, timers, Swiper, GSAP
5. **Tìm DOM trong `scope`** — dùng `scope.find()` thay vì `document.querySelector()` để scope đúng widget instance
6. **Parse options an toàn** — wrap `JSON.parse()` trong try/catch
7. **Đăng ký `onDestroy` callback** — `scope.data('onDestroy', fn)` để page builder gọi khi cleanup
8. **Không dùng ES module** — file JS được include trực tiếp qua `<script>`, không cần `import/export`

## 7. Ví Dụ Hoàn Chỉnh

### Ví dụ 1: Element đơn giản (không có style variant)

**Cấu trúc:**
```
elements/my-widget/
├── my-widget.widget.php
├── views/
│   └── view.blade.php
└── assets/
    └── my-widget.less
```

**my-widget.widget.php:**
```php
<?php

use SkillDo\Cms\Element\Element;
use SkillDo\Cms\Support\Theme;

class MyWidgetElement extends Element
{
    public function __construct()
    {
        parent::__construct('MyWidgetElement', 'My Widget');
        $this->assets('assets/my-widget.less');
    }

    public function icon(): string
    {
        return 'icon-box';
    }

    public function category(): string
    {
        return 'basic';
    }

    public function form(): void
    {
        $this->tabs('generate')->adds(function (\SkillDo\Cms\Form\Form $form)
        {
            $form->text('title', ['label' => 'Tiêu đề', 'language' => true]);
            $form->textarea('description', ['label' => 'Mô tả', 'language' => true]);
            $form->fontIcon('icon', ['label' => 'Icon']);
        });

        $this->tabs('style')->adds(function (\SkillDo\Cms\Form\Form $form)
        {
            $form->addGroup(function (\SkillDo\Cms\Form\Form $form)
            {
                $form->boxBuilding('boxStyle')->popup(false);
            }, $this->groupFormBox('Khung', 'boxStyle', true));

            $form->addGroup(function (\SkillDo\Cms\Form\Form $form)
            {
                $form->textBuilding('titleStyle')->popup(false);
            }, $this->groupFormBox('Tiêu đề', 'titleStyle'));
        });

        parent::form();
    }

    public function widget(): void
    {
        if(empty($this->options->title))
        {
            echo $this->empty();
            return;
        }

        Theme::view($this->getDir().'views/view', [
            'options' => $this->options,
        ]);
    }

    public function default(): void
    {
        $defaults = [
            'title'       => 'Tiêu đề mặc định',
            'description' => 'Mô tả mặc định',
            'icon'        => '<i class="fa-duotone fa-sun"></i>',
        ];

        foreach ($defaults as $key => $value)
        {
            $this->options->{$key} = $this->options->{$key} ?? $value;
        }
    }

    public function cssBuilder(): string
    {
        $this->cssSelector('.item', [
            'data'  => $this->options->boxStyle ?? [],
            'style' => 'box',
        ]);

        $this->cssSelector('.item .title', [
            'data'  => $this->options->titleStyle ?? [],
            'style' => 'text',
        ]);

        return $this->cssBuild();
    }
}
```

**views/view.blade.php:**
```blade
<div class="item">
    @if(!empty($options->icon))
    <div class="icon">{!! $options->icon !!}</div>
    @endif
    <div class="title">{!! $options->title !!}</div>
    @if(!empty($options->description))
    <div class="description">{!! $options->description !!}</div>
    @endif
</div>
```

**assets/my-widget.less:**
```less
.MyWidgetElement .item {
    text-align: center;
    .icon {
        font-size: 32px;
        margin-bottom: 10px;
    }
    .title {
        font-size: 16px;
        font-weight: bold;
    }
    .description {
        font-size: 13px;
        color: #666;
    }
}
```

**Đăng ký trong widget.json:**
```json
{
    "elements": {
        "general": {
            "MyWidgetElement": {
                "path": "widget/elements/my-widget/my-widget.widget.php"
            }
        }
    }
}
```

### Ví dụ 2: Element có style variant

**Cấu trúc:**
```
elements/my-card/
├── style1/
│   ├── my-card-style1.widget.php
│   ├── views/
│   │   └── view.blade.php
│   └── assets/
│       └── my-card-style-1.less
└── style2/
    ├── my-card-style2.widget.php
    ├── views/
    │   └── view.blade.php
    └── assets/
        └── my-card-style-2.less
```

**Tên class cho variant:** `MyCardElementStyle1`, `MyCardElementStyle2`

**Đăng ký widget.json:**
```json
{
    "elements": {
        "general": {
            "MyCardElementStyle1": {
                "path": "widget/elements/my-card/style1/my-card-style1.widget.php"
            },
            "MyCardElementStyle2": {
                "path": "widget/elements/my-card/style2/my-card-style2.widget.php"
            }
        }
    }
}
```

### Ví dụ 3: Element với Repeater

```php
public function form(): void
{
    $this->tabs('generate')->adds(function (\SkillDo\Cms\Form\Form $form)
    {
        $form->repeater('items', [
            'label'  => 'Danh sách',
            'fields' => function (\SkillDo\Cms\Form\Form $form)
            {
                $form->image('image', ['label' => 'Ảnh']);
                $form->text('title', ['label' => 'Tiêu đề', 'language' => true]);
                $form->textarea('content', ['label' => 'Nội dung', 'language' => true]);
            }
        ]);
    });
    parent::form();
}

public function default(): void
{
    $this->options->items = $this->options->items ?? [
        ['image' => '', 'title' => 'Item 1', 'content' => 'Nội dung 1'],
        ['image' => '', 'title' => 'Item 2', 'content' => 'Nội dung 2'],
    ];
}
```

### Ví dụ 4: Element có Ajax (Products)

```php
// Trong widget.json cần thêm:
// "ajax": { "client": "MyProductElement::loadData" }

static function loadData(\SkillDo\Http\Request $request): void
{
    $options = $request->input('options');
    if(hasItems($options))
    {
        // Query data
        $result = ['items' => []];
        // ... build items ...
        response()->success(trans('ajax.load.success'), $result);
    }
    response()->error(trans('ajax.load.error'));
}
```

## 8. Quy Tắc Quan Trọng

### Naming Convention

| Thành phần          | Quy tắc                              | Ví dụ                     |
|---------------------|--------------------------------------|---------------------------|
| Thư mục element     | kebab-case                           | `icon-box`, `text-editor` |
| Thư mục style       | `style` + số                         | `style1`, `style2`        |
| File PHP            | kebab-case + `.widget.php`           | `icon-box.widget.php`     |
| Tên class (đơn)     | PascalCase + `Element`               | `IconBoxElement`          |
| Tên class (variant) | PascalCase + `Element` + `Style` + N | `HeadingElementStyle1`    |
| Constructor key     | Trùng tên class                      | `'IconBoxElement'`        |
| File JS             | kebab-case + `.js`                   | `counter-widget.js`       |
| Class JS            | PascalCase (tự do, khuyến khích giống PHP) | `CounterWidget`     |
| Hook action JS      | `frontend/ready/{TenClassPHP}.default` | `frontend/ready/CounterElement.default` |

### Checklist khi tạo element mới

1. ✅ Tạo thư mục theo đúng cấu trúc (đơn hoặc có style variant)
2. ✅ Tạo file `.widget.php` kế thừa `Element`
3. ✅ Implement các method bắt buộc: `icon()`, `category()`, `form()`, `widget()`
4. ✅ Tạo `default()` để set giá trị mặc định — **bao gồm cả default style** (xem bên dưới)
5. ✅ Tạo `cssBuilder()` nếu có style động
6. ✅ Tạo file `views/view.blade.php`
7. ✅ Tạo file LESS nếu cần (selector gốc = tên class)
8. ✅ Tạo file JS nếu element cần client-side logic (slider, toggle, lazy load, animation)
9. ✅ JS: Đăng ký hook `frontend/ready/{TenClassPHP}.default` + implement `destroy()`
10. ✅ Gọi `parent::form()` cuối method `form()`
11. ✅ Đăng ký trong `widget.json` > `elements` > nhóm phù hợp

### Default Style — Bắt buộc khai báo sẵn trong `default()`

> **Quy tắc:** Mọi field style (`boxStyle`, `itemStyle`, `titleStyle`...) khai báo trong tab `style` **phải có giá trị mặc định** được set trong method `default()`. Điều này đảm bảo khi người dùng kéo element vào trang, panel Kiểu dáng đã hiển thị sẵn giá trị để chỉnh sửa, thay vì để trống.

Giá trị mặc định trong `default()` phải **khớp với CSS hardcode trong file LESS**.

#### Ví dụ: `boxBuilding` — nền + bo góc + shadow + padding

```php
// LESS tương ứng: background:#fff; border-radius:10px; box-shadow:0 2px 16px rgba(0,0,0,.08); padding:16px;
$this->options->boxStyle = $this->options->boxStyle ?? [
    'background' => [
        'active' => 'classic',
        'color'  => ['active' => 'color', 'color' => '#ffffff'],
    ],
    'border' => [
        'style'  => '',
        'radius' => ['top' => '10', 'right' => '10', 'bottom' => '10', 'left' => '10', 'linked' => '1'],
    ],
    'boxShadow' => [
        'color'    => 'rgba(0,0,0,0.08)',
        'x'        => '0',
        'y'        => '2',
        'blur'     => '16',
        'spread'   => '0',
        'position' => 'outline',
    ],
    'padding' => [
        'desktop' => ['top' => '16', 'right' => '16', 'bottom' => '16', 'left' => '16', 'linked' => '1'],
    ],
];
```

#### Ví dụ: `boxBuilding` — nền + viền + bo góc + padding

```php
// LESS tương ứng: background:#f8f9fb; border:1px solid #eef0f4; border-radius:8px; padding:10px 14px;
$this->options->itemStyle = $this->options->itemStyle ?? [
    'background' => [
        'active' => 'classic',
        'color'  => ['active' => 'color', 'color' => '#f8f9fb'],
    ],
    'border' => [
        'style'  => 'solid',
        'width'  => ['top' => '1', 'right' => '1', 'bottom' => '1', 'left' => '1', 'linked' => '1'],
        'color'  => ['active' => 'color', 'color' => '#eef0f4'],
        'radius' => ['top' => '8', 'right' => '8', 'bottom' => '8', 'left' => '8', 'linked' => '1'],
    ],
    'padding' => [
        'desktop' => ['top' => '10', 'right' => '14', 'bottom' => '10', 'left' => '14', 'linked' => '0'],
    ],
];
```

#### Ví dụ: `textBuilding` — font-size + font-weight + màu chữ

```php
// LESS tương ứng: font-size:14px; font-weight:700; color:#111;
$this->options->titleStyle = $this->options->titleStyle ?? [
    'typography' => [
        'fontSize'   => ['desktop' => '14'],
        'fontWeight' => '700',
    ],
    'color' => [
        'active' => 'color',
        'color'  => '#111111',
    ],
];
```

#### Format dữ liệu tham chiếu nhanh

| Field                        | Key chính                                                                                   | Ghi chú                             |
|------------------------------|---------------------------------------------------------------------------------------------|-------------------------------------|
| `background` (classic color) | `['active'=>'classic','color'=>['active'=>'color','color'=>'#hex']]`                        | Dùng trong `boxBuilding.background` |
| `border.style`               | `'solid'`, `'dashed'`, `'dotted'`, `''`                                                     |                                     |
| `border.width`               | `['top'=>'1','right'=>'1','bottom'=>'1','left'=>'1','linked'=>'1']`                         | px                                  |
| `border.color`               | `['active'=>'color','color'=>'#hex']`                                                       | colorBuilding format                |
| `border.radius`              | `['top'=>'8','right'=>'8','bottom'=>'8','left'=>'8','linked'=>'1']`                         | px                                  |
| `boxShadow`                  | `['color'=>'rgba(...)','x'=>'0','y'=>'2','blur'=>'16','spread'=>'0','position'=>'outline']` |                                     |
| `padding/margin`             | `['desktop'=>['top'=>'','right'=>'','bottom'=>'','left'=>'','linked'=>'0']]`                | px                                  |
| `typography.fontSize`        | `['desktop'=>'14']`                                                                         | px                                  |
| `typography.fontWeight`      | `'400'`, `'600'`, `'700'`                                                                   |                                     |
| `color` (text)               | `['active'=>'color','color'=>'#hex']`                                                       | colorBuilding format                |

### Lưu ý

- **`$this->getDir()`** tự động lấy đường dẫn thư mục chứa file PHP hiện tại
- **`$this->options`** là object chứa dữ liệu form, luôn kiểm tra `??` trước khi dùng
- **`$this->empty()`** trả về placeholder HTML khi element chưa có dữ liệu
- **`parent::form()`** phải được gọi cuối cùng trong method `form()`
- LESS selector gốc **phải trùng tên class** (key) để CSS scope đúng
- Dùng CSS variables cho responsive thay vì media queries trong PHP
- **`'language' => true`** trong form field cho phép nhập đa ngôn ngữ
- **`->display('inline')`** hiển thị field trên cùng dòng
- **`->start(N)`** set số cột grid (tổng 12) cho field
- **`->popup(false)`** hiển thị building field inline thay vì popup
- **`->condition('field', [values])`** ẩn/hiện field theo điều kiện
