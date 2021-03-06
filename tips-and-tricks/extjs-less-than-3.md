---
description: Some ExtJs tips
---

# ExtJS &lt;3

## Open backend application with link

With /backend?app=**\[NAME\]** can you open any Module \(/backend/?app=Article\), you can also pass supported variables like /backend/?app=\[**NAME**\]&params\[**NAME**\]=**VALUE** \(/backend/?app=Article&params\[articleId\]=1\)

## Create a real checkbox in config.xml

```markup
<element type="boolean">
    <name>test</name>
    <label lang="de">test</label>
    <label lang="en">test</label>
    <options>
      <xtype>checkbox</xtype>
    </options>
</element>
```

## Use components from another backend application

When you want to use components from an another application, you have to load first the App. Otherwise the Ext.Loader cannot find the right controller to load the files.

**Example for the Voucher Store**

```javascript
Ext.require('Shopware.apps.Voucher');
Ext.create('Shopware.apps.Voucher.store.Voucher', .....);
```

## DatePicker gives null back

Due to an Extjs Hack from Shopware it can sometimes happen that the DatePicker always returns null. 

[https://github.com/shopware/shopware/blob/5.4/engine/Library/ExtJs/overrides/Ext.form.Base.js\#L56](https://github.com/shopware/shopware/blob/5.4/engine/Library/ExtJs/overrides/Ext.form.Base.js#L56,) there "getValues" is called - but in the original ExtJS Form Base "getFieldValues" is called.

### Solution

Overwrite method **getSubmitData**, to return the value from **getModelData.**

Example for Shopware.model.Container

```javascript
applyDateFieldConfig: function () {
    var field = this.callParent(arguments);

    field.getSubmitData = function () {
        return this.getModelData();
    };

    return field;
}
```

## Emotion Element with static combobox values

You can't pass store values in to a combobox in an emotion element like in config.xml. This example will show you how to create a custom extjs store with static values.

{% code-tabs %}
{% code-tabs-item title="Test.php" %}
```php
<?php

namespace Test;
use Shopware\Components\Plugin;
use Shopware\Components\Plugin\Context\InstallContext;

class Test extends Plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'Enlight_Controller_Action_PostDispatch_Backend_Emotion' => 'onPostDispatchEmotion'
        ];
    }
    
    public function onPostDispatchEmotion(\Enlight_Event_EventArgs $args)
    {
        /** @var \Shopware_Controllers_Backend_Emotion $subject */
        $subject = $args->getSubject();
        if ($subject->Request()->getActionName() == 'index') {
            $subject->View()->addTemplateDir($this->getPath() . '/Resources/views');
            $subject->View()->extendsTemplate('emotionstore.js');
        }
    }
    
    public function install(InstallContext $context)
    {
        parent::install($context); // TODO: Change the autogenerated stub
        $emotion = $this->container->get('shopware.emotion_component_installer');
        $vimeoElement = $emotion->createOrUpdate(
            $this->getName(),
            'Vimeo Video',
            [
                'name' => 'Vimeo Video',
                'template' => 'emotion_vimeo',
                'cls' => 'emotion-vimeo-element',
                'description' => 'A simple vimeo video element for the shopping worlds.'
            ]
        );
        $vimeoElement->createComboBoxField([
            'fieldLabel' => 'Test',
            'name' => 'bla',
            'supportText' => 'yay',
            'allowBlank' => false,
            'store' => 'Shopware.apps.Emotion.store.MyTestStore',
            'queryMode' => 'local',
            'displayField' => 'name',
            'valueField' => 'id'
        ]);
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="Resources/views/emotionstore.js" %}
```javascript
//{block name="backend/Emotion/app" append}
Ext.define('Shopware.apps.Emotion.store.MyTestStore', {
    extend: 'Ext.data.Store',
    fields: [
        {
            name: 'id',
            type: 'integer'
        },
        {
            name: 'name',
            type: 'string'
        }
    ],
    data: [
        {
            id: 1,
            name: "LOL"
        },
        {
            id: 2,
            name: "Test"
        }
    ]
})
//{/block}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Show in every form the id of the record

Open developer console and insert following code and open the module.

```javascript
Ext.override(Ext.form.Panel, {
    addedIdField: false,
    getForm: function () {
        var me = this,
            insertComponent = this,
            form = me.callParent(arguments),
            orgMethod = form.loadRecord;

        form.loadRecord = function (record) {
            if (!me.addedIdField && record.get('id')) {
                if (fieldSet = me.down('fieldset')) {
                    insertComponent = fieldSet;
                }

                if (insertComponent.items.items[0].$className === 'Ext.container.Container' || insertComponent.items.items[0].superclass.$className === 'Ext.container.Container') {
                    insertComponent = insertComponent.items.items[0];
                }

                insertComponent.insert(0, {
                    fieldLabel: 'ID',
                    xtype: 'displayfield',
                    name: 'id'
                });
                me.addedIdField = true;
            }

            return orgMethod.apply(this, arguments);
        }

        return form;
    },
});
```

