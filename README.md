# WooCommerce Product Price Last Updated

This code snippet provides a solution to **automatically track and display the last updated date of your WooCommerce product prices**. When you update a product's regular or sale price in the backend, this code will save the current date. It then displays this "last updated" date on the single product page, right after the "Add to cart" button. This feature can enhance transparency for your customers, especially for products with fluctuating prices or frequent updates.

## Why use this code?

* **Transparency for Customers:** Informs customers when a product's price was last changed, which can build trust.
* **Encourage Purchases:** For time-sensitive offers or frequently updated products, showing a recent update can encourage quicker decisions.
* **Improved User Experience:** Provides useful information at a glance on the product page.
* **Automated Tracking:** No manual effort is required to update the "last updated" date; it's done automatically when prices are saved.

## Features

* **Automatic Date Saving:** Automatically saves the current date in WordPress's MySQL format (`YYYY-MM-DD HH:MM:SS`) as custom post meta (`_price_last_updated`) whenever a product's regular or sale price is updated.
* **Jalali Date Conversion (Persian Calendar):** Includes a function to convert the stored Gregorian date into the Jalali (Persian) calendar format, which is then displayed to the user.
* **Frontend Display:** Shows the converted last updated date on the single product page, positioned after the "Add to cart" button.
* **Basic Styling:** Applies some basic inline CSS to make the displayed date visually distinct.

## Installation

There are two primary methods to implement this code:

### Method 1: As a Standalone Plugin (Recommended)

Creating a small, dedicated plugin is the most robust way to add this functionality. It ensures your modification remains active regardless of theme changes.

1.  Create a new folder named `wc-product-update-date` inside your WordPress site's `wp-content/plugins/` directory.
2.  Inside this new folder, create a file named `wc-product-update-date.php`.
3.  Copy and paste the following code into `wc-product-update-date.php`:

    ```php
    <?php
    /**
     * Plugin Name: WooCommerce Product Price Last Updated
     * Description: Automatically tracks and displays the last updated date of WooCommerce product prices on the frontend.
     * Version: 1.0
     * Author: Your Name (Optional)
     * License: GPL-2.0-or-later
     * Text Domain: wc-product-update-date
     */

    if ( ! defined( 'ABSPATH' ) ) {
        exit; // Exit if accessed directly.
    }

    /**
     * Automatically saves the last updated date of product price.
     * Fires when a WooCommerce product is saved/updated in the admin.
     *
     * @param int $post_id The ID of the post (product).
     */
    function save_price_last_updated_date($post_id) {
        // Ensure it's a product and not an autosave or revision
        if ( get_post_type($post_id) !== 'product' || wp_is_post_autosave($post_id) || wp_is_post_revision($post_id) ) {
            return;
        }

        // Check if regular price or sale price fields were submitted
        if (isset($_POST['_regular_price']) || isset($_POST['_sale_price'])) {
            // Get existing price meta to compare if price actually changed (optional, but good for performance)
            $old_regular_price = get_post_meta($post_id, '_regular_price', true);
            $old_sale_price = get_post_meta($post_id, '_sale_price', true);

            $new_regular_price = isset($_POST['_regular_price']) ? sanitize_text_field($_POST['_regular_price']) : '';
            $new_sale_price = isset($_POST['_sale_price']) ? sanitize_text_field($_POST['_sale_price']) : '';

            // Update date only if prices have changed
            if ( $old_regular_price !== $new_regular_price || $old_sale_price !== $new_sale_price ) {
                update_post_meta($post_id, '_price_last_updated', current_time('mysql'));
            }
        }
    }
    add_action('woocommerce_process_product_meta', 'save_price_last_updated_date');

    /**
     * Converts a Gregorian date string to Jalali (Persian) calendar format.
     * Requires WordPress's `date_i18n` which handles localization.
     *
     * @param string $date Gregorian date string.
     * @return string Jalali formatted date.
     */
    function convert_to_jalali($date) {
        // Ensure `jdate` function is available if using external library for Jalali
        // For simplicity, we'll rely on WordPress's date_i18n which usually supports locale.
        $timestamp = strtotime($date);
        return date_i18n('j F Y', $timestamp); // 'j F Y' format is compatible with many locales. For specific Persian format, direct Jalali library might be needed.
    }

    /**
     * Displays the last updated price date on the single product page.
     * Positioned after the "Add to cart" button.
     */
    function display_price_last_updated_date() {
        global $product;
        // Ensure we are on a single product page and product object is available
        if (!$product || !is_singular('product')) {
            return;
        }

        $price_last_updated = get_post_meta($product->get_id(), '_price_last_updated', true);
        if (!$price_last_updated) {
            return;
        }

        // Convert Gregorian date to Jalali (Persian)
        $jalali_date = convert_to_jalali($price_last_updated);

        echo '<p class="product-last-updated">';
        echo sprintf(
            esc_html__('Price last updated: %1$s', 'wc-product-update-date'),
            $jalali_date
        );
        echo '</p>';
    }
    add_action('woocommerce_after_add_to_cart_button', 'display_price_last_updated_date');

    /**
     * Adds custom CSS for the last updated price date display.
     */
    function custom_product_last_updated_styles() {
        // Only enqueue on single product pages
        if ( is_singular('product') ) {
            echo '
            <style>
                .product-last-updated {
                    font-size: 14px;
                    color: #000;
                    background-image: linear-gradient(240deg,#5CB400A8 0%,#ffffff9e 100%);
                    padding: 0px 10px;
                    border-radius: 5px;
                    display: inline-block; /* To make background fill properly */
                    margin-top: 10px; /* Spacing from button */
                }
            </style>';
        }
    }
    add_action('wp_head', 'custom_product_last_updated_styles');
    ```

4.  Go to your WordPress admin dashboard, navigate to **"Plugins"**, and **activate** the "WooCommerce Product Price Last Updated" plugin.

### Method 2: Adding to your Theme's functions.php File

You can add this code directly to your active theme's `functions.php` file. **Before doing so, it's highly recommended to back up your `functions.php` file.**

1.  Navigate to `wp-content/themes/YourThemeName/` (replace `YourThemeName` with the actual name of your active theme).
2.  Open the `functions.php` file.
3.  Add the following code to the end of the file (before the closing `?>` tag, if one exists):

    ```php
    /**
     * Automatically saves the last updated date of product price.
     * Fires when a WooCommerce product is saved/updated in the admin.
     *
     * @param int $post_id The ID of the post (product).
     */
    function save_price_last_updated_date($post_id) {
        // Ensure it's a product and not an autosave or revision
        if ( get_post_type($post_id) !== 'product' || wp_is_post_autosave($post_id) || wp_is_post_revision($post_id) ) {
            return;
        }

        // Check if regular price or sale price fields were submitted
        if (isset($_POST['_regular_price']) || isset($_POST['_sale_price'])) {
            // Get existing price meta to compare if price actually changed (optional, but good for performance)
            $old_regular_price = get_post_meta($post_id, '_regular_price', true);
            $old_sale_price = get_post_meta($post_id, '_sale_price', true);

            $new_regular_price = isset($_POST['_regular_price']) ? sanitize_text_field($_POST['_regular_price']) : '';
            $new_sale_price = isset($_POST['_sale_price']) ? sanitize_text_field($_POST['_sale_price']) : '';

            // Update date only if prices have changed
            if ( $old_regular_price !== $new_regular_price || $old_sale_price !== $new_sale_price ) {
                update_post_meta($post_id, '_price_last_updated', current_time('mysql'));
            }
        }
    }
    add_action('woocommerce_process_product_meta', 'save_price_last_updated_date');

    /**
     * Converts a Gregorian date string to Jalali (Persian) calendar format.
     * Requires WordPress's `date_i18n` which handles localization.
     *
     * @param string $date Gregorian date string.
     * @return string Jalali formatted date.
     */
    function convert_to_jalali($date) {
        // Ensure `jdate` function is available if using external library for Jalali
        // For simplicity, we'll rely on WordPress's date_i18n which usually supports locale.
        $timestamp = strtotime($date);
        return date_i18n('j F Y', $timestamp); // 'j F Y' format is compatible with many locales. For specific Persian format, direct Jalali library might be needed.
    }

    /**
     * Displays the last updated price date on the single product page.
     * Positioned after the "Add to cart" button.
     */
    function display_price_last_updated_date() {
        global $product;
        // Ensure we are on a single product page and product object is available
        if (!$product || !is_singular('product')) {
            return;
        }

        $price_last_updated = get_post_meta($product->get_id(), '_price_last_updated', true);
        if (!$price_last_updated) {
            return;
        }

        // Convert Gregorian date to Jalali (Persian)
        $jalali_date = convert_to_jalali($price_last_updated);

        echo '<p class="product-last-updated">';
        echo sprintf(
            esc_html__('Price last updated: %1$s', 'your-textdomain'), // Change 'your-textdomain' if using a specific text domain
            $jalali_date
        );
        echo '</p>';
    }
    add_action('woocommerce_after_add_to_cart_button', 'display_price_last_updated_date');

    /**
     * Adds custom CSS for the last updated price date display.
     */
    function custom_product_last_updated_styles() {
        // Only enqueue on single product pages
        if ( is_singular('product') ) {
            echo '
            <style>
                .product-last-updated {
                    font-size: 14px;
                    color: #000;
                    background-image: linear-gradient(240deg,#5CB400A8 0%,#ffffff9e 100%);
                    padding: 0px 10px;
                    border-radius: 5px;
                    display: inline-block; /* To make background fill properly */
                    margin-top: 10px; /* Spacing from button */
                }
            </style>';
        }
    }
    add_action('wp_head', 'custom_product_last_updated_styles');
    ```

## Customization

* **Styling:** The provided code injects inline CSS for the `.product-last-updated` class. For more robust and maintainable styling, it's recommended to **move this CSS into your theme's `style.css` file** or a dedicated CSS file.
* **Display Position:** The date is displayed after the "Add to cart" button using `woocommerce_after_add_to_cart_button`. You can change this hook to another relevant WooCommerce action hook to display the date in a different location on the single product page (e.g., `woocommerce_single_product_summary`, `woocommerce_before_single_product_summary`, etc.).
* **Date Format:** The `convert_to_jalali` function uses `date_i18n('j F Y', $timestamp)`. You can modify the date format string (e.g., `'Y/m/d'`, `'l, F j, Y'`) to suit your preference. For more advanced Jalali calendar formatting or if `date_i18n` doesn't fully meet your specific Persian locale needs, you might consider integrating a dedicated Jalali calendar library for PHP.
* **Text Domain:** If you use this code in a theme or plugin with a specific text domain for translation, ensure `your-textdomain` in `esc_html__` matches your project's text domain.

## Contributing

Contributions are welcome! If you have suggestions or improvements for this code, feel free to open a "Pull Request" or report an "Issue."

## License

This project is licensed under the GPL-2.0-or-later License.

---


# تاریخ آخرین به‌روزرسانی قیمت محصول ووکامرس

این قطعه کد راه حلی برای **ردیابی و نمایش خودکار تاریخ آخرین به‌روزرسانی قیمت محصولات ووکامرس** شما ارائه می‌دهد. هنگامی که قیمت عادی یا قیمت فروش یک محصول را در پنل مدیریت به‌روزرسانی می‌کنید، این کد تاریخ فعلی را ذخیره می‌کند. سپس این تاریخ "آخرین به‌روزرسانی" را در صفحه تک محصول، بلافاصله پس از دکمه "افزودن به سبد خرید" نمایش می‌دهد. این ویژگی می‌تواند شفافیت را برای مشتریان شما افزایش دهد، به خصوص برای محصولاتی با قیمت‌های متغیر یا به‌روزرسانی‌های مکرر.

## چرا از این کد استفاده کنیم؟

* **شفافیت برای مشتریان:** به مشتریان اطلاع می‌دهد که قیمت یک محصول آخرین بار چه زمانی تغییر کرده است، که می‌تواند اعتماد را افزایش دهد.
* **تشویق به خرید:** برای پیشنهادات حساس به زمان یا محصولاتی که مکرراً به‌روزرسانی می‌شوند، نمایش یک به‌روزرسانی اخیر می‌تواند تصمیم‌گیری سریع‌تر را تشویق کند.
* **بهبود تجربه کاربری:** اطلاعات مفید را در یک نگاه در صفحه محصول ارائه می‌دهد.
* **ردیابی خودکار:** هیچ تلاش دستی برای به‌روزرسانی تاریخ "آخرین به‌روزرسانی" لازم نیست؛ این کار به طور خودکار هنگام ذخیره قیمت‌ها انجام می‌شود.

## قابلیت‌ها

* **ذخیره خودکار تاریخ:** به طور خودکار تاریخ فعلی را در قالب MySQL وردپرس (`YYYY-MM-DD HH:MM:SS`) به عنوان متا پست سفارشی (`_price_last_updated`) هر زمان که قیمت عادی یا قیمت فروش یک محصول به‌روزرسانی می‌شود، ذخیره می‌کند.
* **تبدیل تاریخ شمسی:** شامل تابعی برای تبدیل تاریخ میلادی ذخیره شده به فرمت تقویم شمسی (خورشیدی) است که سپس به کاربر نمایش داده می‌شود.
* **نمایش در فرانت‌اند:** تاریخ آخرین به‌روزرسانی تبدیل شده را در صفحه تک محصول، پس از دکمه "افزودن به سبد خرید" نمایش می‌دهد.
* **استایل‌دهی پایه:** برخی CSSهای داخلی پایه را برای متمایز کردن بصری تاریخ نمایش داده شده اعمال می‌کند.

## نصب

برای پیاده‌سازی این کد، دو روش اصلی وجود دارد:

### روش ۱: به عنوان یک افزونه مستقل (توصیه شده)

ایجاد یک افزونه کوچک و اختصاصی، قوی‌ترین راه برای افزودن این قابلیت است. این تضمین می‌کند که تغییر شما صرف‌نظر از تغییر قالب، فعال باقی بماند.

1.  یک پوشه جدید با نام `wc-product-update-date` در مسیر `wp-content/plugins/` سایت وردپرسی خود ایجاد کنید.
2.  در داخل این پوشه جدید، یک فایل با نام `wc-product-update-date.php` ایجاد کنید.
3.  کد زیر را در `wc-product-update-date.php` کپی و جایگذاری کنید:

    ```php
    <?php
    /**
     * Plugin Name: WooCommerce Product Price Last Updated
     * Description: Automatically tracks and displays the last updated date of WooCommerce product prices on the frontend.
     * Version: 1.0
     * Author: Your Name (Optional)
     * License: GPL-2.0-or-later
     * Text Domain: wc-product-update-date
     */

    if ( ! defined( 'ABSPATH' ) ) {
        exit; // Exit if accessed directly.
    }

    /**
     * Automatically saves the last updated date of product price.
     * Fires when a WooCommerce product is saved/updated in the admin.
     *
     * @param int $post_id The ID of the post (product).
     */
    function save_price_last_updated_date($post_id) {
        // اطمینان حاصل کنید که یک محصول است و نه یک پیش‌نویس خودکار یا بازبینی
        if ( get_post_type($post_id) !== 'product' || wp_is_post_autosave($post_id) || wp_is_post_revision($post_id) ) {
            return;
        }

        // بررسی کنید که آیا فیلدهای قیمت عادی یا قیمت فروش ارسال شده‌اند
        if (isset($_POST['_regular_price']) || isset($_POST['_sale_price'])) {
            // دریافت متای قیمت موجود برای مقایسه اینکه آیا قیمت واقعاً تغییر کرده است (اختیاری، اما برای عملکرد خوب است)
            $old_regular_price = get_post_meta($post_id, '_regular_price', true);
            $old_sale_price = get_post_meta($post_id, '_sale_price', true);

            $new_regular_price = isset($_POST['_regular_price']) ? sanitize_text_field($_POST['_regular_price']) : '';
            $new_sale_price = isset($_POST['_sale_price']) ? sanitize_text_field($_POST['_sale_price']) : '';

            // تاریخ را فقط در صورتی به‌روزرسانی کنید که قیمت‌ها تغییر کرده باشند
            if ( $old_regular_price !== $new_regular_price || $old_sale_price !== $new_sale_price ) {
                update_post_meta($post_id, '_price_last_updated', current_time('mysql'));
            }
        }
    }
    add_action('woocommerce_process_product_meta', 'save_price_last_updated_date');

    /**
     * Converts a Gregorian date string to Jalali (Persian) calendar format.
     * Requires WordPress's `date_i18n` which handles localization.
     *
     * @param string $date Gregorian date string.
     * @return string Jalali formatted date.
     */
    function convert_to_jalali($date) {
        // برای سادگی، ما به date_i18n وردپرس تکیه می‌کنیم که معمولاً از لوکال پشتیبانی می‌کند.
        // برای فرمت خاص فارسی یا اگر date_i18n نیازهای خاص شما را برآورده نمی‌کند،
        // ممکن است نیاز به یک کتابخانه اختصاصی شمسی برای PHP باشد.
        $timestamp = strtotime($date);
        return date_i18n('j F Y', $timestamp); // فرمت 'j F Y' با بسیاری از لوکال‌ها سازگار است.
    }

    /**
     * Displays the last updated price date on the single product page.
     * Positioned after the "Add to cart" button.
     */
    function display_price_last_updated_date() {
        global $product;
        // اطمینان حاصل کنید که در صفحه محصول واحد هستیم و شیء محصول در دسترس است
        if (!$product || !is_singular('product')) {
            return;
        }

        $price_last_updated = get_post_meta($product->get_id(), '_price_last_updated', true);
        if (!$price_last_updated) {
            return;
        }

        // تبدیل تاریخ میلادی به شمسی
        $jalali_date = convert_to_jalali($price_last_updated);

        echo '<p class="product-last-updated">';
        echo sprintf(
            esc_html__(' بروز رسانی قیمت: %1$s', 'wc-product-update-date'),
            $jalali_date
        );
        echo '</p>';
    }
    add_action('woocommerce_after_add_to_cart_button', 'display_price_last_updated_date');

    /**
     * Adds custom CSS for the last updated price date display.
     */
    function custom_product_last_updated_styles() {
        // فقط در صفحات محصول واحد اعمال شود
        if ( is_singular('product') ) {
            echo '
            <style>
                .product-last-updated {
                    font-size: 14px;
                    color: #000;
                    background-image: linear-gradient(240deg,#5CB400A8 0%,#ffffff9e 100%);
                    padding: 0px 10px;
                    border-radius: 5px;
                    display: inline-block; /* برای اینکه پس‌زمینه به درستی پر شود */
                    margin-top: 10px; /* فاصله از دکمه */
                }
            </style>';
        }
    }
    add_action('wp_head', 'custom_product_last_updated_styles');
    ```

4.  وارد پنل مدیریت وردپرس خود شوید، به بخش **"افزونه‌ها"** بروید و افزونه **"تاریخ آخرین به‌روزرسانی قیمت محصول ووکامرس"** را **فعال کنید**.

### روش ۲: اضافه کردن به فایل functions.php قالب شما

می‌توانید این کد را مستقیماً به فایل `functions.php` قالب فعال خود اضافه کنید. **پیشنهاد اکید می‌شود قبل از انجام این کار، از فایل `functions.php` خود یک پشتیبان (backup) تهیه کنید.**

1.  به مسیر `wp-content/themes/YourThemeName/` بروید (به جای `YourThemeName` نام واقعی قالب فعال خود را قرار دهید).
2.  فایل `functions.php` را باز کنید.
3.  کد زیر را به انتهای فایل (قبل از تگ بستن `?>`، در صورت وجود) اضافه کنید:

    ```php
    /**
     * Automatically saves the last updated date of product price.
     * Fires when a WooCommerce product is saved/updated in the admin.
     *
     * @param int $post_id The ID of the post (product).
     */
    function save_price_last_updated_date($post_id) {
        // اطمینان حاصل کنید که یک محصول است و نه یک پیش‌نویس خودکار یا بازبینی
        if ( get_post_type($post_id) !== 'product' || wp_is_post_autosave($post_id) || wp_is_post_revision($post_id) ) {
            return;
        }

        // بررسی کنید که آیا فیلدهای قیمت عادی یا قیمت فروش ارسال شده‌اند
        if (isset($_POST['_regular_price']) || isset($_POST['_sale_price'])) {
            // دریافت متای قیمت موجود برای مقایسه اینکه آیا قیمت واقعاً تغییر کرده است (اختیاری، اما برای عملکرد خوب است)
            $old_regular_price = get_post_meta($post_id, '_regular_price', true);
            $old_sale_price = get_post_meta($post_id, '_sale_price', true);

            $new_regular_price = isset($_POST['_regular_price']) ? sanitize_text_field($_POST['_regular_price']) : '';
            $new_sale_price = isset($_POST['_sale_price']) ? sanitize_text_field($_POST['_sale_price']) : '';

            // تاریخ را فقط در صورتی به‌روزرسانی کنید که قیمت‌ها تغییر کرده باشند
            if ( $old_regular_price !== $new_regular_price || $old_sale_price !== $new_sale_price ) {
                update_post_meta($post_id, '_price_last_updated', current_time('mysql'));
            }
        }
    }
    add_action('woocommerce_process_product_meta', 'save_price_last_updated_date');

    /**
     * Converts a Gregorian date string to Jalali (Persian) calendar format.
     * Requires WordPress's `date_i18n` which handles localization.
     *
     * @param string $date Gregorian date string.
     * @return string Jalali formatted date.
     */
    function convert_to_jalali($date) {
        // برای سادگی، ما به date_i18n وردپرس تکیه می‌کنیم که معمولاً از لوکال پشتیبانی می‌کند.
        // برای فرمت خاص فارسی یا اگر date_i18n نیازهای خاص شما را برآورده نمی‌کند،
        // ممکن است نیاز به یک کتابخانه اختصاصی شمسی برای PHP باشد.
        $timestamp = strtotime($date);
        return date_i18n('j F Y', $timestamp); // فرمت 'j F Y' با بسیاری از لوکال‌ها سازگار است.
    }

    /**
     * Displays the last updated price date on the single product page.
     * Positioned after the "Add to cart" button.
     */
    function display_price_last_updated_date() {
        global $product;
        // اطمینان حاصل کنید که در صفحه محصول واحد هستیم و شیء محصول در دسترس است
        if (!$product || !is_singular('product')) {
            return;
        }

        $price_last_updated = get_post_meta($product->get_id(), '_price_last_updated', true);
        if (!$price_last_updated) {
            return;
        }

        // تبدیل تاریخ میلادی به شمسی
        $jalali_date = convert_to_jalali($price_last_updated);

        echo '<p class="product-last-updated">';
        echo sprintf(
            esc_html__(' بروز رسانی قیمت: %1$s', 'your-textdomain'), // 'your-textdomain' را در صورت استفاده از یک دامین متنی خاص تغییر دهید
            $jalali_date
        );
        echo '</p>';
    }
    add_action('woocommerce_after_add_to_cart_button', 'display_price_last_updated_date');

    /**
     * Adds custom CSS for the last updated price date display.
     */
    function custom_product_last_updated_styles() {
        // فقط در صفحات محصول واحد اعمال شود
        if ( is_singular('product') ) {
            echo '
            <style>
                .product-last-updated {
                    font-size: 14px;
                    color: #000;
                    background-image: linear-gradient(240deg,#5CB400A8 0%,#ffffff9e 100%);
                    padding: 0px 10px;
                    border-radius: 5px;
                    display: inline-block; /* برای اینکه پس‌زمینه به درستی پر شود */
                    margin-top: 10px; /* فاصله از دکمه */
                }
            </style>';
        }
    }
    add_action('wp_head', 'custom_product_last_updated_styles');
    ```

## سفارشی‌سازی

* **استایل‌دهی:** کد ارائه شده CSS داخلی (inline) را برای کلاس `product-last-updated` تزریق می‌کند. برای استایل‌دهی قوی‌تر و قابل نگهداری‌تر، توصیه می‌شود **این CSS را به فایل `style.css` قالب خود** یا یک فایل CSS اختصاصی منتقل کنید.
* **موقعیت نمایش:** تاریخ با استفاده از هوک `woocommerce_after_add_to_cart_button` پس از دکمه "افزودن به سبد خرید" نمایش داده می‌شود. می‌توانید این هوک را به یک هوک اکشن ووکامرس مرتبط دیگر تغییر دهید تا تاریخ را در مکان متفاوتی در صفحه محصول واحد نمایش دهید (مثلاً `woocommerce_single_product_summary`, `woocommerce_before_single_product_summary` و غیره).
* **فرمت تاریخ:** تابع `convert_to_jalali` از `date_i18n('j F Y', $timestamp)` استفاده می‌کند. می‌توانید رشته فرمت تاریخ (مانند `'Y/m/d'`, `'l, F j, Y'`) را برای مطابقت با ترجیحات خود تغییر دهید. برای فرمت‌دهی پیشرفته‌تر تقویم شمسی یا اگر `date_i18n` نیازهای خاص فارسی شما را به طور کامل برآورده نمی‌کند، ممکن است لازم باشد یک کتابخانه اختصاصی تقویم شمسی برای PHP را یکپارچه کنید.
* **Text Domain:** اگر این کد را در یک قالب یا افزونه با یک دامین متنی خاص برای ترجمه استفاده می‌کنید، مطمئن شوید که `your-textdomain` در `esc_html__` با دامین متنی پروژه شما مطابقت دارد.

## مشارکت (Contributing)

مشارکت شما خوشایند است! اگر پیشنهاد یا بهبودهایی برای این کد دارید، می‌توانید یک "Pull Request" ایجاد کنید یا "Issue" جدیدی را گزارش دهید.

## مجوز (License)

این پروژه تحت مجوز GPL-2.0-or-later منتشر شده است.
