---
category: Develop
---
# Parameters

Each tag, trigger, and variable can optionally define one or multiple parameters. These parameters have the same API as [Plugin settings](/guides/plugin-settings).

## Specifying parameters

Here is an example on how to define a parameter which you can later access in the JavaScript part of a tag, trigger, or variable.

```php
public function getParameters()
{
    return array(
        $this->makeSetting($name = 'popupText', $default = 'This is the default value', FieldConfig::TYPE_STRING, function (FieldConfig $field) {
            $field->title = 'Popup Text';
            $field->uiControl = FieldConfig::UI_CONTROL_TEXTAREA;
            $field->description = 'Please Enter the text ';
            $field->validators[] = new NotEmpty();
        })
    );
}
```

You may notice a couple of things such as the parameter name `popupText` under which the parameter will be
later available in Javscript using `parameters.get('popupText')`, the default value, the title that will be shown in the UI to the user, which UI control should be used, and more.

### UI controls

You can choose between a selection of different UI controls, for example a check box or a select field. Check out the [FieldConfig API](https://developer.matomo.org/api-reference/Piwik/Settings/FieldConfig) for more information on the available
control types.

### Selecting variables

In the Tag Manager you may optionally allow a user to choose a variable instead of writing only a hard coded value. To do this,
simply set the template file to one of these values:

```php
// a text field that allows the user to choose a variable
$field->uiControl = FieldConfig::UI_CONTROL_TEXT;
$field->customUiControlTemplateFile = self::FIELD_TEMPLATE_VARIABLE;

// a textarea that allows the user to choose a variable
$field->uiControl = FieldConfig::UI_CONTROL_TEXTAREA;
$field->customUiControlTemplateFile = self::FIELD_TEMPLATE_TEXTAREA_VARIABLE;

// requires the user to select a "MatomoConfigurationVariable"
$field->customUiControlTemplateFile = self::FIELD_TEMPLATE_VARIABLE_TYPE;
$field->uiControlAttributes = array('variableType' => 'MatomoConfiguration');
```

### Showing only specific parameters

In your template, you may want to show certain parameters only when a different parameter has a specific value. In the below example the user can select a "tracking type". When the selected type is "Goal", we want to show another field which lets the
user enter a "Goal ID". To achieve this, you can specify a `condition` property. In the below example this is `trackingType == "goal"`. It means the parameter will be only shown when the user has selected the tracking type `goal`.

```php
public function getParameters()
{
    $trackingType = $this->makeSetting('trackingType', 'pageview', FieldConfig::TYPE_STRING, function (FieldConfig $field) {
        $field->title = 'Tracking Type';
        $field->uiControl = FieldConfig::UI_CONTROL_SINGLE_SELECT;
        $field->availableValues = array(
            'pageview' => 'Pageview',
            'goal' => 'Goal',
        );
    });

    return array(
        $trackingType,
        $this->makeSetting('idGoal', '', FieldConfig::TYPE_INT, function (FieldConfig $field) use ($trackingType) {
            $field->title = 'Goal ID';
            $field->condition = 'trackingType == "goal"';
            if ($trackingType->getValue() === 'goal') {
                $field->validators[] = new NotEmpty();
                $field->validators[] = new CharacterLength(1, 500);
            }
        }),
    );
```

### Validating a value

You have two ways to validate a value:

* Use one of many predefined validators such as `NotEmpty`, `CharacterLength`, `Email`, `NumberRange`:

```php
$field->validators[] = new Piwik\Validators\NotEmpty();
$field->validators[] = new Piwik\Validators\Email();
$field->validators[] = new Piwik\Validators\UrlLike();
$field->validators[] = new Piwik\Validators\CharacterLength($min = 1, $max = 500);
$field->validators[] = new Piwik\Validators\NumberRange($min = 1, $max = 500);
```

* Define a custom `validate` callable like this:

```php
$field->validate = function ($value, Setting $setting) {
    if ($value > 60) {
        throw new \Exception('The time limit is not allowed to be greater than 60 minutes.');
    }
}
```

You may also use both ways within one field. Any `validator` will be executed before the custom `validate` method.

### Transforming a value

Before a value is saved, you may want to transform a value to ensure the value has the expected format. For example, you may want to ensure a URL always has a protocol set like this:

```php
$field->transform = function ($value, Setting $setting) {
    if (strpos($value, 'http') === false) {
        $value = 'https://' . $value;
    }

    return $value;
}
```

### More information

To get more information, check out the [FieldConfig API](https://developer.matomo.org/api-reference/Piwik/Settings/FieldConfig).
