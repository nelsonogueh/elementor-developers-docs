# Add New Field Type

The forms widget has builtin field types but it also accepts new fields registered by external developers.

## Registering Field Types

To register new field types you just need to initiate the field class:

```php
function register_new_form_fields() {

	require_once( __DIR__ . '/forms/fields/field-1.php' );
	require_once( __DIR__ . '/forms/fields/field-2.php' );

	\ElementorPro\Plugin::instance()->modules_manager->get_modules( 'forms' )->add_form_field_type( 'field-1', new \Field_1() );
	\ElementorPro\Plugin::instance()->modules_manager->get_modules( 'forms' )->add_form_field_type( 'field-2', new \Field_2() );

}
add_action( 'elementor_pro/init', 'register_new_form_fields' );
```

## Field Types Class

To create your own field type, you need to extend the `Field_Base` abstract class:

```php
class Elementor_Test_Control extends \ElementorPro\Modules\Forms\Fields\Field_Base {
}
```

## Field Types Methods

A simple field type skeleton class will look as follows:

```php
class Elementor_Test_Field_Type extends \ElementorPro\Modules\Forms\Fields\Field_Base {

	public $depended_scripts = [];

	public $depended_styles = [];

	public function get_type() {}

	public function get_name() {}

	public function render() {}

	public function validation() {}

}
```

* **Field Type** – The `get_type()` method returns the field name that will be used in the code.

* **Field Name** – The `get_title()` method returns the field label that will be displayed to the user.

* **Field Render** – The `render()` method renders the data and display the field output.

* **Field Render** – The `validation()` method runs a series of checks to ensure the data complies to certain rules.

## Example Field Type

Lets create a new field type for Elementor form widget. The field will allow end-users to enter their credit card number.

```php
<?php
class Elementor_Credit_Card_Number_Field_Type extends \ElementorPro\Modules\Forms\Fields\Field_Base {

	public function get_type() {
		return 'credit card number';
	}

	public function get_name() {
		return __( 'Credit Card Number', 'plugin-name' );
	}

	public function render( $item, $item_index, $form ) {
		$form_id = $form->get_id();

		$form->add_render_attribute(
			'input' . $item_index,
			[
				'class' => 'elementor-field-textual',
				'for' => $form_id . $item_index,
				'type' => 'tel',
				'inputmode' => 'numeric',
				'maxlength' => '19',
				'pattern' => '[0-9\s]{19}',
				'placeholder' => $item['placeholder'],
				'autocomplete' => 'cc-number',
			]
		);

		echo '<input ' . $form->get_render_attribute_string( 'input' . $item_index ) . '>';
	}

	public function validation( $field, $record, $ajax_handler ) {
		if ( empty( $field['value'] ) ) {
			return;
		}

		if ( preg_match( '/^[0-9]{4}\s[0-9]{4}\s[0-9]{4}\s[0-9]{4}$/', $field['value'] ) !== 1 ) {
			$ajax_handler->add_error( $field['id'], __( 'Credit card number must be in "XXXX XXXX XXXX XXXXX" format.', 'plugin-name' ) );
		}
	}

	public function update_controls( $widget ) {
		$elementor = \ElementorPro\Plugin::elementor();

		$control_data = $elementor->controls_manager->get_control_from_stack( $widget->get_unique_name(), 'form_fields' );

		if ( is_wp_error( $control_data ) ) {
			return;
		}

		$field_controls = [
			'placeholder' => [
				'name' => 'placeholder',
				'label' => __( 'Placeholder', 'plugin-name' ),
				'type' => \Elementor\Controls_Manager::TEXT,
				'default' => 'xxxx xxxx xxxx xxxx',
				'condition' => [
					'field_type' => $this->get_type(),
				],
			],
		];

		$control_data['fields'] = $this->inject_field_controls( $control_data['fields'], $field_controls );

		$widget->update_control( 'form_fields', $control_data );
	}

}
```