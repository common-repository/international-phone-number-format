=== International Phone Number Format ===
Contributors: endisha
Tags: phone, number, international, format, mask, woocommerce, wordpress
Requires at least: 6.0
Requires PHP: 8.0
Tested up to: 6.3
Stable tag: 1.0.0
License: GPLv2 or later
License URI: https://www.gnu.org/licenses/gpl-2.0.html

The International Phone Number Format plugin allows you to effortlessly format and validate international phone numbers within WooCommerce and WordPress fields.

== Description ==

The International Phone Number Format plugin empowers you to seamlessly format, validate, and store international phone numbers according to the E.164 standard. This functionality spans across various sections of your WooCommerce-powered online store and within WordPress itself. Notably, the plugin effectively formats and validates phone numbers in key areas such as WooCommerce checkout fields, customer account address fields, order details within the WordPress admin dashboard, and user edits page in the WordPress admin dashboard.

The plugin dynamically adapts to the countries associated with billing and shipping addresses in your WooCommerce settings and designates them as the countries for phone number formatting fields.

= Features =

* Enables easy configuration of fields through the settings.
* Automatically formats and validates phone numbers in WooCommerce checkout fields.
* Automatic country detection based on the user's IP address.
* Supports automatic formatting and validation of phone numbers in customer account addresses.
* Ensures automatic formatting and validation of phone numbers in Dashboard order details.
* Provides automatic formatting and validation of phone numbers in Dashboard edit-user details.
* Allows customization by setting specific fields as phone numbers for ease of use.
* Offers the flexibility to set additional custom Regex patterns for specific country codes using filters.
* Stores phone numbers in the E.164 standard ([+][country code][phone number]) for consistency and compatibility.

= Requirements =

* WordPress 5.7 or newer.
* WooCommerce 6.0 or newer.
* PHP version 8.0 or newer.

== Installation ==

= Minimum Requirements =

* PHP 8.0 or greater is recommended
* MySQL 5.6 or greater is recommended

= Automatic installation =

Automatic installation is the easiest option -- WordPress will handles the file transfer, and you won’t need to leave your web browser. To do an automatic install of International Phone Number Format, log in to your WordPress dashboard, navigate to the Plugins menu, and click “Add New.”
 
In the search field type “International Phone Number Format,” then click “Search Plugins.” Once you’ve found us,  you can view details about it such as the point release, rating, and description. Most importantly of course, you can install it by! Click “Install Now,” and WordPress will take it from there.

= Manual installation =

Manual installation method requires downloading the International Phone Number Format plugin and uploading it to your web server via your favorite FTP application. The WordPress codex contains [instructions on how to do this here](https://wordpress.org/support/article/managing-plugins/#manual-plugin-installation).


== Frequently Asked Questions ==

= How do I enable and configure the international phone number format? =

Once the plugin is activated, navigate to WooCommerce > Settings > International Phone Number Format tab in your WordPress admin dashboard.

= Can I add new fields? =

Yes, you can add new fields using the `intl_phone_number_format_fields` filter.

= How do I add a new custom WooCommerce checkout field? = 

Certainly. if you wish to add a Shipping Phone number field named `shipping_phone` to the checkout fields, it will be automatically formatted using the international phone number mask. If the field is set as required, it will undergo validation both in the frontend and backend as a valid number. Here's how you can achieve that:
`
<?php
add_filter('woocommerce_shipping_fields', function ($fields)
{
	$fields['shipping_phone']['type'] = 'tel';
	$fields['shipping_phone']['label'] = __('Shipping Phone Number', 'woocommerce');
	$fields['shipping_phone']['class'] = array('form-row-wide');
	$fields['shipping_phone']['required'] = false;
	return $fields;
}, 10, 1);
`

= How do I add a new custom field? =

Of course, let's say you want to add a "Mobile Phone Number" field to user account details in WooCommerce:

Add the custom field using the `intl_phone_number_format_fields` filter. Here's an example code snippet:

`
<?php
add_filter('intl_phone_number_format_fields', function ($fields){
    $fields[] = array(
        'id' => 'mobile',
        'enable' => true,
        'desc' => 'Mobile phone number on the profile page',
        'label' => 'Mobile',
        'frontend_validation' => true, // JS frontend validation
        'backend_validation' => true,  // Backend validation
        'countries' => 'all', // Countries related to fields (all, billing, or shipping)
        'type' => 'custom', // Field type (custom, billing, or shipping)
    );
    return $fields;
});
`

Display the added field in the frontend "My Account" details section. Use the following code:

`
<?php
add_action('woocommerce_edit_account_form_start', function () {
	$user = wp_get_current_user();
	?>
		<p class="woocommerce-form-row woocommerce-form-row--wide form-row form-row-wide">
			<label for="mobile"><?php _e('Mobile phone', 'woocommerce'); ?>
				<span class="required">*</span></label>
			<input type="text" class="woocommerce-Input woocommerce-Input--phone input-text" name="mobile" id="mobile" value="<?php echo esc_attr($user->mobile); ?>" />
		</p>
	<?php
});
`

Implement validation for the field. Use the woocommerce_save_account_details_errors action hook along with the intl validation service:

`
<?php
add_action('woocommerce_save_account_details_errors', function (WP_Error $errors) {
	$fields = (new IPNFP_Fields_Service)->get_fields();
	$validated = (new IPNFP_Validate_Service)->validate($fields);
	if (!empty($validated)) {
		foreach ($validated as $field => $error) {
			$errors->add($field, $error);
		}
	}
}, 10, 1);
`

Save the user's input for the field:

`
<?php
add_action('woocommerce_save_account_details', function (int $user_id) {
	if (isset($_POST['mobile'])) {
		$mobile = sanitize_text_field($_POST['mobile']);
		update_user_meta($user_id, 'mobile', $mobile);
	}
}, 10, 1);
`

In case the JS file is not included, you can include it using:

`
<?php
add_filter('intl_phone_number_format_validate_enqueue_js', function ($valid){
    if (is_account_page()) {
        $valid = true;
    }
    return $valid;
});
`

By following these steps, the field should be included and formatted with the International Phone Number Format mask.

= How to Add Additional Validation for Specific Country Numbers =

You have the flexibility to add custom validation using regular expressions for specific countries. If you need to enforce a specific format for phone numbers in a particular country, you can achieve this by the intl_phone_number_format_custom_country_prefixes_validation filter.

Here's an example scenario: Let's assume you want to apply custom validation rules for Libyan phone numbers, allowing only a specific format. In this case, you can utilize the following filter to associate the country calling code with the regular expression pattern:

`
<?php
// The structure should follow this format:
// 'calling code' => 'regular expression pattern'
add_filter('intl_phone_number_format_custom_country_prefixes_validation', function ($patterns) {
	$patterns['+218'] = '^\+2189[1-6][0-9]{7}$';
	return $patterns;
});
`

= How do I modify fields? =

The plugin provides several filters that allow you to modify the fields, including built-in fields such as `billing_phone`, `_billing_phone`, `shipping_phone`, and `_shipping_phone`, as well as custom fields that you added using `intl_phone_number_format_fields` filter.


This filter allows you to enable or disable the field.

`intl_phone_number_format_modify_{FIELD}_enable`

This filter allows you to enable or disable JS frontend validation for the specified field.

`intl_phone_number_format_modify_{FIELD}_frontend_validation`

This filter to enable or disable backend validation for the specified field.

`intl_phone_number_format_modify_{FIELD}_backend_validation`

This filter to adjust the countries that will be shown as calling codes for the field. Accepted values are: `all`, `billing`, and `shipping`.

`intl_phone_number_format_modify_{FIELD}_countries`

This filter to define the field type. For example, if the field is related to billing, set it as `billing`. If it's for shipping, set it as `shipping`. Otherwise, set it as `custom`.

`intl_phone_number_format_modify_{FIELD}_type`


== Changelog ==
= 1.0.0 2023-11-25 =
* Initial release.

== Upgrade Notice ==
= 1.0.0 =
This is the initial release of the International Phone Number Format plugin.

== Screenshots ==
1. Plugin Settings
2. Checkout Fields
3. Address Fields
4. Order Details

== External Library Usage ==

This plugin utilizes the [intl-tel-input](https://github.com/jackocnr/intl-tel-input) library to provide phone number formatting functionality.

== License ==
International Phone Number Format is licensed under the GNU General Public License v2 or later.
