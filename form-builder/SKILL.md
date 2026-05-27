---
name: form-builder
description: Hướng dẫn sử dụng Form Builder trong SkillDo CMS v8 - tạo form, các loại field và dữ liệu lưu trữ.
---

# Form Builder - SkillDo CMS v8

> **Namespace:** `SkillDo\Cms\Form\Form`
> **Helper:** `form()`

## Tạo Form

```php
use SkillDo\Cms\Form\Form;
$form = new Form();
// hoặc
$form = form();
```

### Cấu hình Form

```php
$form->setFormId('my_form_id');
$form->setIsUpload(true);       // Form upload file
$form->setIsValid(true);        // Bật JS validation
$form->setCallbackValidJs('myHandler'); // Callback sau validation
```

### Hiển thị Form

```php
echo $form->open('post', [
    'validation' => true,
    'callback'   => 'myHandler',
    'class'      => ['my-class'],
    'style'      => ['font-size' => '18px'],
    'file'       => true,
]);
echo $form->html();
echo $form->close();
```

## Thêm Field

```php
// Cách 1: Phương thức add
$form->add('field_name', 'text', ['label' => 'Label'], $value);

// Cách 2: Phương thức động
$form->text('field_name', ['label' => 'Label'], $value);
```

### Args chung cho tất cả field

| Param | Type | Mô tả |
|-------|------|-------|
| `label` | string | Label của field |
| `start` | string/int | HTML đầu field hoặc số cột (1-12) |
| `end` | string | HTML cuối field |
| `note` | string | Ghi chú dưới field |
| `class` | string/array | CSS class |
| `id` | string | ID tùy chỉnh |
| `defaultValue` | mixed | Giá trị mặc định |
| `validations` | Rule | Quy tắc validation |
| `condition` | array | Điều kiện hiển thị |

## Danh Sách Field

---

### 1. Text / Password / Email / Number / Tel / URL / Phone

**Type:** `text`, `password`, `email`, `number`, `tel`, `url`, `phone`

```php
$form->text('username', ['label' => 'Tên']);
$form->password('pass', ['label' => 'Mật khẩu']);
$form->email('email', ['label' => 'Email']);
$form->number('age', ['label' => 'Tuổi', 'min' => 18]);
$form->tel('phone', ['label' => 'SĐT']);
$form->url('website', ['label' => 'Website']);
```

**Dữ liệu lưu:** `string` — chuỗi văn bản đơn giản.

---

### 2. Hidden

**Type:** `hidden`

```php
$form->hidden('token', [], 'value_here');
```

**Dữ liệu lưu:** `string` — giá trị ẩn.

---

### 3. Textarea

**Type:** `textarea`

```php
$form->textarea('note', ['label' => 'Ghi chú']);
```

**Dữ liệu lưu:** `string` — văn bản nhiều dòng.

---

### 4. Wysiwyg / WysiwygShort

**Type:** `wysiwyg`, `wysiwyg-short`

```php
$form->wysiwyg('content', ['label' => 'Nội dung']);
$form->wysiwygShort('summary', ['label' => 'Tóm tắt']);
```

**Dữ liệu lưu:** `string` — HTML content.

**Thư viện cần nhúng (theme):** `tinymce`

---

### 5. Select

**Type:** `select`

```php
$form->select('status', ['label' => 'Trạng thái'])->options([
    'active' => 'Hoạt động',
    'inactive' => 'Không hoạt động',
]);
```

**Dữ liệu lưu:** `string` — giá trị option được chọn.

---

### 6. Select2

**Type:** `select2`

```php
$form->select2('tags', ['label' => 'Tags'])->options($options)->multiple(true);
```

**Dữ liệu lưu:** `string` (single) hoặc `array` (multiple).

**Thư viện cần nhúng (theme):** `select2`

---

### 7. Checkbox

**Type:** `checkbox`

```php
$form->checkbox('features', ['label' => 'Tính năng'])->options([
    'seo' => 'SEO', 'cache' => 'Cache',
]);
```

**Dữ liệu lưu:** `array` — danh sách các value được chọn.

---

### 8. Checkbox Icon

**Type:** `checkbox-icon`

```php
$form->checkboxIcon('styles', ['label' => 'Style'])->options([
    'italic' => ['label' => 'Italic', 'icon' => '<i class="fa-light fa-italic"></i>'],
]);
```

**Dữ liệu lưu:** `array` — danh sách value được chọn.

---

### 9. Checkbox Tree

**Type:** `checkbox-tree`

```php
$form->checkboxTree('categories', ['label' => 'Danh mục', 'cate_type' => 'post_categories']);
```

**Dữ liệu lưu:** `array` — danh sách ID danh mục được chọn.

---

### 10. Radio

**Type:** `radio`

```php
$form->radio('gender', ['label' => 'Giới tính'])->options([
    'male' => 'Nam', 'female' => 'Nữ',
]);
```

**Dữ liệu lưu:** `string` — giá trị option được chọn.

---

### 11. Radio Icon

**Type:** `radio-icon`

```php
$form->radioIcon('align', ['label' => 'Align'])->options([
    'left' => ['label' => 'Trái', 'icon' => '<i class="fa-light fa-align-left"></i>'],
    'center' => ['label' => 'Giữa', 'icon' => '<i class="fa-light fa-align-center"></i>'],
]);
```

**Dữ liệu lưu:** `string` — giá trị option được chọn.

---

### 12. Switch (On/Off)

**Type:** `switch`

```php
$form->switch('is_active', ['label' => 'Kích hoạt']);
// Custom values
$form->switch('status', ['label' => 'Status'])->options([0 => 'off', 1 => 'on']);
// Custom labels
$form->switch('status', ['label' => 'Status', 'label-true' => 'Bật', 'label-false' => 'Tắt']);
```

**Dữ liệu lưu:** `int|string` — mặc định `1` (bật) / `0` (tắt), hoặc custom value.

---

### 13. Color

**Type:** `color`

```php
$form->color('text_color', ['label' => 'Màu chữ']);
```

**Dữ liệu lưu:** `string` — mã màu (hex, rgb, rgba).

**Thư viện cần nhúng (theme):** `@melloware/coloris`

---

### 14. Image

**Type:** `image`

```php
$form->image('thumbnail', ['label' => 'Ảnh đại diện']);
```

**Dữ liệu lưu:** `string` — đường dẫn file ảnh.

---

### 15. File

**Type:** `file`

```php
$form->file('document', ['label' => 'Tài liệu']);
```

**Dữ liệu lưu:** `string` — đường dẫn file.

---

### 16. Video

**Type:** `video`

```php
$form->video('intro_video', ['label' => 'Video']);
```

**Dữ liệu lưu:** `string` — đường dẫn video.

---

### 17. Date / Time / Datetime

**Type:** `date`, `time`, `datetime`

```php
$form->date('start_date', ['label' => 'Ngày bắt đầu']);
$form->time('start_time', ['label' => 'Giờ']);
$form->datetime('event_at', ['label' => 'Thời gian']);
```

**Dữ liệu lưu:**
- `date`: `string` format `dd/mm/yyyy`
- `time`: `string` format `HH:mm`
- `datetime`: `string` format `dd/mm/yyyy HH:mm`

**Thư viện cần nhúng (theme):** `air-datepicker`

---

### 18. Date Range

**Type:** `daterange`

```php
$form->daterange('period', ['label' => 'Khoảng thời gian']);
```

**Dữ liệu lưu:** `string` format `dd/mm/yyyy - dd/mm/yyyy`

**Thư viện cần nhúng (theme):** `daterangepicker` + `moment`

---

### 19. Range

**Type:** `range`

```php
$form->range('opacity', ['label' => 'Độ mờ'])->min(0)->max(100);
```

**Dữ liệu lưu:** `int|float` — giá trị số trong khoảng min-max.

---

### 20. Price

**Type:** `price`

```php
$form->price('product_price', ['label' => 'Giá']);
```

**Dữ liệu lưu:** `string` — số có dấu phẩy ngàn (ví dụ: `1,000,000`).

---

### 21. Font Icon

**Type:** `font-icon`

```php
$form->fontIcon('icon', ['label' => 'Chọn icon']);
```

**Dữ liệu lưu:** `string` — class CSS của icon (ví dụ: `fa-light fa-home`).

---

### 22. Repeater

**Type:** `repeater`

```php
$form->repeater('items', ['label' => 'Danh sách'])->fields(function ($repeater) {
    $repeater->text('title', ['label' => 'Tiêu đề', 'start' => 6]);
    $repeater->image('image', ['label' => 'Ảnh', 'start' => 6]);
    $repeater->textarea('desc', ['label' => 'Mô tả', 'start' => 12]);
});
```

**Dữ liệu lưu:** `array` — mảng các item, mỗi item chứa các field con:
```php
[
    'unique_id_1' => ['title' => '...', 'image' => '...', 'desc' => '...'],
    'unique_id_2' => ['title' => '...', 'image' => '...', 'desc' => '...'],
]
```

---

### 23. Select Tabs (Tab)

**Type:** `tab`

```php
$form->tab('layout', ['label' => 'Layout'])->options([
    'grid' => 'Grid', 'list' => 'List',
]);
```

**Dữ liệu lưu:** `string` — giá trị tab được chọn.

---

### 24. Select Image

**Type:** `select-img`

```php
$form->selectImg('template', ['label' => 'Template'])->options([
    'style1' => ['label' => 'Style 1', 'img' => 'path/to/img.png'],
    'style2' => ['label' => 'Style 2', 'img' => 'path/to/img2.png'],
]);
```

**Dữ liệu lưu:** `string` — value của option được chọn.

---

### 25. Numeric Selector

**Type:** `numericSelector`

```php
$form->numericSelector('columns', ['label' => 'Số cột', 'min' => 1, 'max' => 12]);
```

**Dữ liệu lưu:** `int` — số trong khoảng min-max.

---

### 26. Container

**Type:** `container`

```php
$form->container('layout', ['label' => 'Bố cục']);
```

**Dữ liệu lưu:** `string` — một trong: `full`, `container`, `in-container`.

---

### 27. Gallery (Select thư viện)

**Type:** `gallery`

```php
$form->gallery('album', ['label' => 'Chọn thư viện']);
```

**Dữ liệu lưu:** `int` — ID của gallery.

---

### 28. Gallery Item (Thư viện ảnh)

**Type:** `gallery-item`

```php
$form->galleryItem('images', ['label' => 'Danh sách ảnh']);
```

**Dữ liệu lưu:** `array` — danh sách đường dẫn ảnh.

---

### 29. Menu

**Type:** `menu`

```php
$form->menu('main_menu', ['label' => 'Chọn menu']);
```

**Dữ liệu lưu:** `int` — ID của menu.

---

### 30. Page

**Type:** `page`

```php
$form->page('about_page', ['label' => 'Chọn trang']);
```

**Dữ liệu lưu:** `int` — ID của trang.

---

### 31. Post

**Type:** `posts`

```php
$form->post('featured', ['label' => 'Bài viết', 'post_type' => 'post']);
```

**Dữ liệu lưu:** `int` — ID bài viết.

---

### 32. Post Category

**Type:** `post-category`

```php
$form->postCategory('category', ['label' => 'Danh mục', 'cate_type' => 'post_categories']);
```

**Dữ liệu lưu:** `int` — ID danh mục.

---

### 33. Post Category Tree

**Type:** `post-category-tree`

```php
$form->postCategoryTree('cats', ['label' => 'Danh mục', 'cate_type' => 'post_categories']);
```

**Dữ liệu lưu:** `array` — danh sách ID danh mục được check.

---

### 34. Popover Advance

**Type:** `popover-advance`

```php
// Chọn page
$form->popoverAdvance('pages', ['label' => 'Chọn trang', 'search' => 'page']);
// Chọn post
$form->popoverAdvance('posts', ['label' => 'Chọn bài', 'search' => 'post', 'taxonomy' => 'post']);
// Chọn category
$form->popoverAdvance('cats', ['label' => 'Danh mục', 'search' => 'category']);
// Chọn user
$form->popoverAdvance('users', ['label' => 'User', 'search' => 'user']);
// Custom popover
$form->popoverAdvance('items', ['label' => 'Custom', 'search' => 'myPopover', 'multiple' => true]);
```

**Dữ liệu lưu:** `int` (single) hoặc `array` (multiple) — ID đối tượng được chọn.

---

### 35. Input Responsive

**Type:** `input-responsive`

```php
$form->inputResponsive('font_size', ['label' => 'Font Size']);
```

**Dữ liệu lưu:**
```php
['desktop' => '', 'tablet' => '', 'mobile' => '']
```

---

### 36. Input Dimension

**Type:** `input-dimension`

```php
$form->inputDimension('margin', ['label' => 'Margin']);
```

**Dữ liệu lưu:**
```php
['top' => 0, 'right' => 0, 'bottom' => 0, 'left' => 0]
```

---

### 37. Input Dimension Responsive

**Type:** `input-dimension-responsive`

```php
$form->inputDimensionResponsive('padding', ['label' => 'Padding']);
```

**Dữ liệu lưu:**
```php
[
    'desktop' => ['top' => 0, 'right' => 0, 'bottom' => 0, 'left' => 0],
    'tablet'  => ['top' => 0, 'right' => 0, 'bottom' => 0, 'left' => 0],
    'mobile'  => ['top' => 0, 'right' => 0, 'bottom' => 0, 'left' => 0],
]
```

---

### 38. Flexbox

**Type:** `flexbox`

```php
$form->flexBox('layout', ['label' => 'Flexbox']);
```

**Dữ liệu lưu:**
```php
[
    'direction' => 'column',       // row|column|row-reverse|column-reverse
    'justify_content' => 'start',  // start|center|end|between|around|evenly
    'align_items' => '',           // start|center|end|stretch
    'gap' => '',                   // số px
    'wrap' => '',                  // nowrap|wrap
]
```

---

### 39. Typography

**Type:** `typography`

```php
$form->typography('heading_typo', ['label' => 'Typography']);
```

**Dữ liệu lưu:**
```php
[
    'fontFamily' => '',
    'fontSize' => ['desktop' => '', 'tablet' => '', 'mobile' => ''],
    'fontWeight' => '',
    'lineHeight' => '',
    'align' => '',             // left|center|right|justify
    'textStyle' => [],         // ['italic', 'underline', 'uppercase']
]
```

**customInput options:** `fontFamily`, `fontSize`, `fontSizeResponsive`, `textStyle`, `align`, `lineHeight`, `fontWeight`

---

### 40. Spacing

**Type:** `spacing`

```php
$form->spacing('element_spacing', ['label' => 'Spacing']);
```

**Dữ liệu lưu:**
```php
[
    'margin' => [
        'desktop' => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
        'tablet'  => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
        'mobile'  => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
    ],
    'padding' => [
        'desktop' => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
        'tablet'  => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
        'mobile'  => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
    ],
]
```

**CSS helper:** `Template::cssSpacing($data)`
**customInput:** `padding` (bool), `margin` (bool)

---

### 41. Background

**Type:** `background`

```php
$form->background('bg', ['label' => 'Background']);
```

**Dữ liệu lưu:**
```php
[
    'color' => '',
    'gradientUse' => '0',
    'gradientColor1' => '', 'gradientColor2' => '',
    'gradientType' => 'linear',
    'gradientRadialDirection1' => 'center', 'gradientRadialDirection2' => '180',
    'gradientPositionStart' => '0', 'gradientPositionEnd' => '100',
    'image' => '', 'imageSize' => 'cover', 'imagePosition' => 'center center',
]
```

**CSS helper:** `Template::cssBg($data)`

---

### 42. Border

**Type:** `border`

```php
$form->border('box_border', ['label' => 'Border']);
```

**Dữ liệu lưu:**
```php
[
    'style' => '',
    'color' => '',
    'width' => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
    'radius' => ['top'=>'','right'=>'','bottom'=>'','left'=>''],
]
```

**CSS helper:** `Template::cssBorder($data)`
**customInput:** `border` (bool), `radius` (bool)

---

### 43. Box Shadow

**Type:** `box-shadow`

```php
$form->boxShadow('shadow', ['label' => 'Shadow']);
```

**Dữ liệu lưu:**
```php
['color'=>'', 'x'=>'', 'y'=>'', 'blur'=>'', 'spread'=>'', 'position'=>'outline']
```

**CSS helper:** `Template::cssBoxShadow($data)`

---

### 44. Text Building

**Type:** `text-building`

```php
$form->textBuilding('title_style', ['label' => 'Title Style', 'customInput' => [
    'txtInput' => true, 'colorHover' => false,
]]);
```

**Dữ liệu lưu:**
```php
[
    'txt' => '', 'color' => '', 'fontFamily' => '0',
    'fontSize' => ['desktop'=>'','tablet'=>'','mobile'=>''],
    'lineHeight' => '0',
    'margin' => /* inputDimensionResponsive */,
    'padding' => /* inputDimensionResponsive */,
    'stroke' => ['color'=>'', 'width'=>''],
    'shadow' => ['color'=>'', 'x'=>'', 'y'=>'', 'blur'=>''],
]
```

**CSS helper:** `Template::cssText($data)`
**customInput:** `txtInput`, `fontFamily`, `fontSize`, `fontSizeResponsive`, `fontWeight`, `lineHeight`, `textStyle`, `color`, `colorHover`, `align`, `margin`, `padding`, `stroke`, `shadow`, `tabAdvanced`

---

### 45. Color Building

**Type:** `color-building`

```php
$form->colorBuilding('link_color', ['label' => 'Màu link']);
```

**Dữ liệu lưu:**
```php
['color' => '', 'colorHover' => '']
```

---

### 46. Box Building

**Type:** `box-building`

```php
$form->boxBuilding('card', ['label' => 'Card Style']);
```

**Dữ liệu lưu:**
```php
[
    'background' => /* background data */,
    'border' => /* border data */,
    'boxShadow' => /* boxShadow data */,
    'margin' => /* inputDimensionResponsive */,
    'padding' => /* inputDimensionResponsive */,
    'hover' => ['borderColor'=>'', 'background'=> /* background data */],
]
```

**CSS helper:** `Template::cssBox($data)`
**customInput:** `background`, `border`, `margin`, `padding`, `boxShadow`, `hover`

---

### 47. Button Building

**Type:** `button-building`

```php
$form->buttonBuilding('btn_style', ['label' => 'Button Style']);
```

**Dữ liệu lưu:**
```php
[
    'background' => /* background data */,
    'border' => /* border data */,
    'color' => '', 'fontFamily' => '0',
    'fontSize' => ['desktop'=>'','tablet'=>'','mobile'=>''],
    'lineHeight' => '0',
    'hover' => ['color'=>'', 'borderColor'=>'', 'background'=> /* background data */],
    'margin' => /* inputDimensionResponsive */,
    'padding' => /* inputDimensionResponsive */,
    'boxShadow' => /* boxShadow data */,
]
```

**CSS helper:** `Template::cssButton($data)`
**customInput:** `background`, `border`, `color`, `fontFamily`, `fontSize`, `textStyle`, `lineHeight`, `align`, `fontWeight`, `margin`, `padding`, `boxShadow`, `hover`

---

### 48. None (HTML)

Chèn HTML tùy ý vào form (không lưu dữ liệu):

```php
$form->none('<div class="my-html">Custom HTML</div>');
```

---

### 49. Heading / Widget Heading

Hiển thị tiêu đề phân vùng (không lưu dữ liệu):

```php
$form->heading('section_title', ['label' => 'Cấu hình chung']);
$form->widgetHeading('widget_title', ['label' => 'Widget']);
```

---

### 50. Form Field

**Type:** `form-field`

Tạo giao diện kéo thả để xây dựng form động:

```php
$form->formField('contact_fields', ['label' => 'Form Fields']);
```

**Dữ liệu lưu:** `array` — mảng field configs:
```php
[
    'id1' => [
        'name'=>'', 'label'=>'', 'placeholder'=>'',
        'type'=>'text', 'column'=>'100',
        'columnTablet'=>'', 'columnMobile'=>'',
        'required'=>'0', 'options'=>'',
        'min'=>'', 'max'=>'',
        'id'=>'', 'class'=>'', 'attributes'=>'',
    ],
]
```

---

## Group và Layout

```php
// Nhóm field
$form->addGroup(function ($group) {
    $group->text('field1', ['label' => 'Field 1', 'start' => 6]);
    $group->text('field2', ['label' => 'Field 2', 'start' => 6]);
}, ['start' => 12, 'label' => 'Group Label']);

// Tab
$form->addTab(function ($tab) {
    $tab->text('name', ['label' => 'Tên']);
}, 'tab-general', ['active' => true]);

// Responsive wrapper
$form->addResponsive('field', ['type' => 'text', 'label' => 'Field']);
```

## Validation

```php
use SkillDo\Validate\Rule;

$form->text('email', [
    'label' => 'Email',
    'validations' => Rule::make()->notEmpty()->email()
]);

// Xác thực request
$validate = request()->validate($form);
if ($validate->fails()) {
    $errors = $validate->errors();
}
```

### Các Rule có sẵn

| Rule | Mô tả |
|------|-------|
| `notEmpty($trim)` | Không rỗng |
| `email()` | Email hợp lệ |
| `numeric()` | Là số |
| `integer()` | Số nguyên |
| `string()` | Là chuỗi |
| `alpha()` | Chỉ chữ cái |
| `alphaDash()` | Chữ, số, `-`, `_` |
| `alphaNum()` | Chữ và số |
| `min($min)` | Giá trị tối thiểu |
| `max($max)` | Giá trị tối đa |
| `between($min,$max)` | Trong khoảng |
| `date($format)` | Ngày hợp lệ |
| `url()` | URL hợp lệ |
| `phone($countries)` | SĐT hợp lệ |
| `color()` | Mã màu hợp lệ |
| `unique($table,$col)` | Duy nhất trong DB |
| `identical($data,$type)` | Giống giá trị khác |
| `in($array)` | Nằm trong danh sách |
| `custom($closure)` | Tùy chỉnh |

## Thêm Field Mới

### Cách 1: Class (khuyến nghị)

Tạo file trong `core/Form/Field/` (theme/plugin):

```php
namespace PluginMyPlugin\Form\Field;
use SkillDo\Cms\Form\InputBuilder;

class MyField extends InputBuilder {
    function __construct($args = [], mixed $value = null) {
        parent::__construct($args, $value);
        $this->type = 'my-field';
    }

    public function getName(): string { return $this->name; }

    public function output(): static {
        $this->output .= form_input($this->attributes(true));
        return $this;
    }
}
```

Sử dụng:
```php
$form->MyField('name', ['label' => 'Label']);
$form->add('name', 'my-field', ['label' => 'Label']);
```

### Cách 2: Static method

```php
$form->add('name', 'MyClass::field', ['label' => 'Label']);
```

### Cách 3: Function

```php
function _form_my_field($params, $value) { return form_input($params, $value); }
$form->add('name', 'my_field', ['label' => 'Label']);
```
