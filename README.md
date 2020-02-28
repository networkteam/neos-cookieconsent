# Sandstorm CookieConsent

This package helps you create a cookie consent that is more fine-grained than most others.

- Can be configured by the integrator
- All text elements of the popup can be customized by the editor of your site (for the parts that the integrator chooses to expose towards the neos editor through the backend ui)
- Multi-Language compatible (automatic translation of all labels in the modal and customizable texts through Neos content dimensions)
- Easily extensible -> Apps can simply be added and configured as nodetypes


# Compatibility and Maintenance

This package is currently being maintained for Neos 4.3 LTS. It is stable, we use it in our projects.

| Neos / Flow Version        | Sandstorm.CookieConsent Version | Maintained |
|----------------------------|---------------------------------|------------|
| Neos 4.3 LTS, Flow 5.3 LTS | 1.x                             | Yes        |


# Quickstart

## 1. Add your own derived Node Type. 

Here we decided to place the link to your privacy policy in the inspector of the node type of CookieConsent.
We also specify a tab and a group in which it will appear.

```yaml
'Vendor.Site:CookieConsent':
  superTypes:
    'Sandstorm.CookieConsent:CookieConsent': true
  ui:
    inspector:
      tabs:
        general:
          label: Einstellungen
          position: 1
          icon: icon-pencil
      groups:
        settingsPrivacyPolicy:
          label: Datenschutzerklärung
          tab: general
          icon: ellipsis-h
          collapsed: true
  properties:
    privacyPolicyHref:
      type: string
      ui:
        label: Verlinkte Datenschutzseite
        inspector:
          group: settingsPrivacyPolicy
          editor: Neos.Neos/Inspector/Editors/LinkEditor
```

## 2. Add the Node Type somewhere to your page

For example as child node:

```yaml
'Vendor.Site:RootPage':
  superTypes:
    'Neos.Neos:Document': true
  childNodes:
    cookie-consent:
      type: 'Vendor.Site:CookieConsent'
      ui:
        label: Cookie Consent
```

## 3. Add rendering for your Node Type

This package provides you with a rendering component that takes care of loading the javascript and css needed. It also configures the library so it knows about any apps for which you need a consent by the user.

Your fusion component acts as the integrational component, `Sandstorm.CookieConsent:Component.CookieConsent` is the presentational component.

A simplified example:

```
prototype(Vendor.Site:CookieConsent) < prototype(Neos.Fusion:Component) {
    _privacyPolicyHref = /* TODO */
    _privacyPolicyHref.@process.convert = Neos.Neos:ConvertUris
    
    _cookieModalTranslations = /* TODO */
    _language = /* TODO */
    _apps = /* TODO */

    renderer = Sandstorm.CookieConsent:Component.CookieConsent {
        privacyPolicyHref = ${props._privacyPolicyHref}
        cookieModalTranslations = ${props._cookieModalTranslations}
        language = ${props._language}
        apps = ${props._apps}
        includeLibraryCss = true
    }
}
```

## 4. Pass in dynamic values
In this example it was decided to let the editor change the privacy policy link in the Neos inspector.

The example assumes you placed it directly below your root page (= site-node). Make sure to adapt to your node paths depending on where you placed the node.

```
prototype(Vendor.Site:CookieConsent) < prototype(Neos.Fusion:Component) {
    @context._cookieConsentProps = ${q(site).find('cookie-consent').get(0).properties}

    _privacyPolicyHref = ${_cookieConsentProps.privacyPolicyHref}
    _privacyPolicyHref.@process.convert = Neos.Neos:ConvertUris

    _cookieModalTranslations = /* TODO */

    renderer = Sandstorm.CookieConsent:Component.CookieConsent {
        privacyPolicyHref = ${props._privacyPolicyHref ? props._privacyPolicyHref : '/footer/datenschutz.html'}
        cookieModalTranslations = ${props._cookieModalTranslations}
        language = ${site.context.dimensions.language[0] ? site.context.dimensions.language[0] : 'de'}
        apps = ${q(site).children('cookie-consent').children().find('apps').children().get()}
        includeLibraryCss = true
    }
}
```

## 5. Translate the modal (optional)
The cookie consent can be translated. It has reasonable defaults for many languages and will try to determine the language of
 your page automatically (for example by looking at your `<html lang>` attribute. 
 For available translations see [the github repo](https://github.com/KIProtect/klaro/tree/master/src/translations).

Note the fusion key `_cookieModalTranslations`. It reads more properties from the CookieConsent nodetype that we defined in step 1.
One will be added as an example below. You can also provide static strings and not let the editor change the keys in the inspector in the
Neos backend - it's up to you. Here the editor is supposed to provide their own texts.

```
prototype(Vendor.Site:CookieConsent) < prototype(Neos.Fusion:Component) {
    @context._cookieConsentProps = ${q(site).find('cookie-consent').get(0).properties}

    _privacyPolicyHref = ${_cookieConsentProps.privacyPolicyHref}
    _privacyPolicyHref.@process.convert = Neos.Neos:ConvertUris

    _cookieModalTranslations = Neos.Fusion:DataStructure {
        ok = ${_cookieConsentProps.ok}
        decline = ${_cookieConsentProps.decline}
        consentNotice = Neos.Fusion:DataStructure {
            description = ${_cookieConsentProps.consentNoticeDescription}
            learnMore = ${_cookieConsentProps.consentNoticeLearnMore}
        }
        consentModal = Neos.Fusion:DataStructure {
            title = ${_cookieConsentProps.consentModalTitle}
            description = ${_cookieConsentProps.consentModalDescription}
            privacyPolicy = Neos.Fusion:DataStructure {
                name = ${_cookieConsentProps.consentModalPrivacyPolicyName}
                text = ${_cookieConsentProps.consentModalPrivacyPolicyText}
            }
        }
        # Static labels, will not be translated. Hopefully generic enough to be used in any language
        purposes = Neos.Fusion:DataStructure {
            livechat = 'Live Chat'
            styling = 'Styling'
            tracking = 'Tracking'
        }
    }

    renderer = Sandstorm.CookieConsent:Component.CookieConsent {
        privacyPolicyHref = ${props._privacyPolicyHref ? props._privacyPolicyHref : '/footer/datenschutz.html'}
        cookieModalTranslations = ${props._cookieModalTranslations}
        language = ${site.context.dimensions.language[0] ? site.context.dimensions.language[0] : 'de'}
        apps = ${q(site).children('cookie-consent').children().find('apps').children().get()}
        includeLibraryCss = true
    }
}
```

```yaml
'Vendor.Site:CookieConsent':
  superTypes:
    'Sandstorm.CookieConsent:CookieConsent': true
  ui: 
    inspector:
      groups:
        smallPopup:
          label: Kleines Popup
          tab: general
          icon: icon-pencil
          position: 100
          collapsed: true
      /* ... */
  properties:
    ok:
      type: string
      defaultValue: Alle Cookies zulassen
      ui:
        label: Cookies-Zulassen Button
        inspector:
          group: smallPopup
  /* Add all translations that should be translated by the editor ... */
```

The easiest way to find out what you can translate is to change the language of the cookieConsent to something that does not exist,
e.g. "foo" (see in the renderer of the following component, there the language key is passed as a property). In the frontend it 
will then fail to load translations and tell you the path where it was looking for the translation.



## 6. Add apps that need a users consent

The package comes with a few apps (e.g. [Google Analytics](https://github.com/sandstorm/neos-cookieconsent/blob/master/Configuration/NodeTypes.CookieConsent.App.GoogleAnalytics.yaml) liable for consent already preconfigured that you can just add from the Neos backend to the collection of app below your CookieConsent node.

In case you need an app that's not yet included, simply copy one of the [existing node types](https://github.com/sandstorm/neos-cookieconsent/tree/master/Configuration) and fill in the necessary information.

**Please open a pull request or an issue with your apps configuration so others can benefit from it :)**


## 7. Place your new Component in your markup

Render your new component somewhere on your page. 

`body.some.fusion.path.cookieConsent = Vendor.Site:CookieConsent`



# Customization

TODO

## 1. Thing to customize

## 2. Thing to customize




# Next steps / possible further development



# License &amp; Copyright

MIT-Licensed, (c) Sandstorm Media GmbH 2020
