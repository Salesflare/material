@ngdoc content
@name Migration to Angular Material
@description

<style>
  table {
    margin: 24px 2px;
    box-shadow: 0 1px 2px rgba(10, 16, 20, 0.24), 0 0 2px rgba(10, 16, 20, 0.12);
    border-radius: 2px;
    background-color: white;
    color: rgba(0,0,0,0.87);
    border-spacing: 0;
  }
  table thead > {
    vertical-align: middle;
    border-color: inherit;
  }
  table thead > tr {
    vertical-align: inherit;
    border-color: inherit;
  }
  table thead > tr > th {
    background-color: white;
    border-bottom: 1px solid rgba(0, 0, 0, 0.12);
    color: #333;
    font-size: 12px;
    font-weight: 500;
    padding: 8px 24px;
    text-align: left;
    line-height: 31px;
    min-width: 64px;
  }
  table tbody > tr > th,
  table tbody > tr > td {
    border-bottom: 1px solid rgba(0, 0, 0, 0.12);
    padding: 16px;
    text-align: left;
    line-height: 15px;
    vertical-align: top;
  }
</style>

# Migration to Angular Material and the Angular CDK

While AngularJS Material has not yet entered Long Term Support (LTS) mode like [AngularJS has][172],
the Angular Components team's resources are focused on [Angular Material][171] and the Angular
[Component Dev Kit (CDK)][cdk]. 

For applications with long-term development and support plans, consideration should be made for
migration to the latest version of [Angular][aio], Material, and the CDK.

Many of the details, that are specific to the AngularJS to Angular migration, are covered in the
official [ngUpgrade Guide][ngUpgrade]. If you are on a recent version of AngularJS
Material (`v1.1.9+`) and AngularJS, then the combination of this document and the official
ngUpgrade Guide should contain the majority of the information that you need to start planning
your migration.

## Key Concepts

The [ngUpgrade Preparation Guide][173] covers a number of important steps for preparing your
application for the migration.

These include:
- Following the [AngularJS Style Guide][style-guide]
- Using a Module Loader like [Webpack][webpack] as part of your build
- Migrating your application to TypeScript
- Migrating many of your AngularJS directives to Component directives while avoiding use of certain
  attributes

In addition to this preparation work, you should use the content of this document in order to make
migration plans based on how features and components have changed between AngularJS Material and
Angular Material.

The content of the rest of this document includes:
- AngularJS Material features that have moved to the Angular CDK
- Significant changes to the theming system
- The extraction of the AngularJS Material layout system into a separate library
- Alternative, light-weight layout options available in the Angular CDK
- A comparison of AngularJS Material to Angular Material directives and services

### Angular CDK

Some of the features of AngularJS Material have been made more generic and moved into the [Angular
CDK][cdk]. 

These include:

- [`$mdPanel`][109] becomes the CDK's [`Overlay`][110]
- [`md-virtual-repeat`][68] becomes the CDK's [`*cdkVirtualFor`][40] and
  `md-virtual-repeat-container` becomes `cdk-virtual-scroll-viewport`
- [`$mdLiveAnnouncer`][111] moves to the CDK's [`LiveAnnouncer`][112]
- `$mdUtil`'s [`bidi()`][113] function has been expanded to the `Directionality` Service and `Dir` 
  Directive [in the CDK][114]
- [`md-input`][52]'s `md-no-autogrow` and `max-rows` behaviors move into the CDK's
  [`CdkTextareaAutosize`][115]
- The [layout][108] system had significant changes which are discussed in a
  separate [section of this guide](#layout)

### Compile-time vs. Run-time theming

AngularJS Material offers powerful [Theming Features][106]. Many of these features do not exist in
Angular Material. Additionally, while most theming was configured in JavaScript for
AngularJS Material, Angular Material uses [Sass][170] features like variables and mixins to
configure theming.

#### AngularJS Material Compile-time Theming
Features include:

- [Pre-built palettes][150] based on the [Material Design Palettes][148]
- [Custom palette creation][151] including definition of hues and contrast colors
- [Custom theme creation][152] including definition of color intentions and whether it is a dark or
  light theme
- [Generation of multiple themes][153] at build time
- Use specific themes on specific elements ([overriding the default theme][155])
- Modify the [browser header and system status bar colors][149] on mobile at compile-time

#### AngularJS Material Run-time Theming
Features include:

- [Definition and generation of themes][143] at run-time
- [Change the application-wide default theme to another theme][154] at run-time
- [Use dynamically changing themes for specific components][156] (overriding the default theme)
- Modify the [browser header and system status bar colors][149] on mobile at run-time
- [Dynamically apply `theme-palette-hue-opacity` color combinations][73] as CSS styles to your own
  application-specific components

#### Challenges

AngularJS Material's theming system is built on the generation of CSS. As mentioned in our
[Under the hood][157] section of the theming guide, this means that 16 `<style>` tags are added to
the document's `<head>` for each theme. While this system is powerful and dynamic, it stretches the
limitations of modern browsers. If many themes are generated at compile-time, the resulting app will
put a strain on low-bandwidth connections. If many themes are generated at run-time, the resulting
app may generate high CPU and/or memory usage.

Unfortunately, unlike native apps ([Android][158], [Windows][159]), some browsers are not yet
equipped to handle rich theming configurations. There is hope for the future though...

Given the experiences learned from the AngularJS Material theming system, the team decided that
another approach would be required for Angular Material.

Unfortunately, the new approach [CSS Variables][160] (also known as CSS Custom Properties) is not
supported in IE11. Thus Angular Material does not yet implement many of the dynamic, run-time
theming features found in AngularJS Material. When IE11 reaches EOL, or when a CSS Variable
polyfill/ponyfill is deemed acceptable, these features will be re-evaluated.

Angular Material [7.3.3][161] fixed a significant issue that blocked the usage of CSS Variables in
color palettes when defining a theme. [This discussion][162] includes some information about
CSS Variable polyfill/ponyfills and some experimentation that has been done.

#### Angular Material Compile-time Theming
Features include:

- [Pre-built palettes][163] based on the [Material Design Palettes][148]
- Custom palette creation including definition of hues and contrast colors can be accomplished by
  copying one of the [provided palettes][167], modifying it, and then using it in your custom theme
  creation
- [Custom theme creation][164] including definition of color intentions and whether it is a dark or
  light theme
- [Generation of multiple themes][165] at build time
- [Theme customization for a set of component types][166] (i.e. `mat-button` and `mat-checkbox`)
- Use Angular Material themes to [style application or library components][168]

Unsupported features:
- There is no built-in, automatic way to change the theme of a specific Angular Material
element while not affecting other components of that type. For example, one `mat-button` cannot
use the default theme and then another `mat-button` use `myCustomTheme`. Instead you would need to
manually apply the desired theme colors and hues using CSS selectors and the
[`mat-color` mixin][74].

#### Run-time Theming in an Angular Material app

Features include:
- Angular's [Meta Service][147] can be used to dynamically modify `<meta>` tags that affect browser
header and status bar colors. The [AngularJS Material docs][149] for this feature have links to
learn more about the specific `<meta>` tags.

Unsupported features:
- Changing a component's hues like AngularJS Material's [`md-hue-1` class][169]
- Definition and generation of themes at run-time
- Change the application-wide default theme to another theme at run-time
- Use dynamically changing themes for specific components (overriding the default theme) at run-time
- Dynamically apply `theme-palette-hue-opacity` color combinations as CSS styles to your own
  application-specific components


### <a name="layout"></a> Changes to Layout Features

The AngularJS Material [layout][108] system includes includes attributes and classes like `layout`,
`flex`, `show`, `hide`, etc.

#### Angular Layout

This comprehensive layout system was moved into its own project named [Angular Layout][120] 
(`@angular/flex-layout`). 

The package contains the following:
- A [Declarative API][116] that includes attributes like `fxLayout`, `fxFlex`, `fxHide`, `fxShow`,
  and more.
- A [Responsive API][117] that includes attributes like `fxLayout.lt-md`, `fxFlex.gt-sm`,
  `fxHide.gt-xs`, `[ngClass.sm]`, `[ngStyle.xs]`, and more.
- Both APIs include [CSS Grid][65] features in addition to the CSS Flexbox features from
  AngularJS Material.
- The ability to define [Custom Breakpoints][118]
- A [MediaObserver Service][119] that allows subscribing to `MediaQuery` activation changes

The [Angular Layout][120] library's full UMD bundle is around `275` KBs. This size can be reduced by
consuming the various modules and using the Angular CLI to remove unused library code, but this
may be more complexity than you need.

#### CDK Layout

The Angular CDK's [Layout][121] module provides:
- A set of `Breakpoints` that match the [Material Design][122] responsive layout grid breakpoints
- A `BreakpointObserver` Service that provides
    - the ability to check media query matching state via `isMatched()`
    - the ability to subscribe to a stream of events based on changes to an array of media queries
      that are passed into `observe()`
- A `MediaMatcher` Service that provides access to the low level, native [`MediaQueryList`][123]

If you are building responsive components or need responsive features in your application with a
minimal increase in bundle size, then the CDK's Layout module is an option you should consider.

### Typography

AngularJS Material [typography classes][107] are generally identical to the Angular Material
[typography classes][27].

Angular Material also adds the following typography features:
- [Customization via Sass mixins][174] that are similar to the theme configuration mixins
- Utility Sass mixins and functions for use in [styling your custom components][175]
  
## Comparison of Features and Migration Tips

Angular Material introduces a number of [new components](#new-components) and features that do not
exist in AngularJS Material. Angular Material also makes greater use of semantic HTML and native
elements to improve accessibility.

However, some directives and services have not been implemented in Angular Material. These are
indicated by an `N/A` in the New Docs column of the feature tables found below. 

The Notes column, in each row of the tables, includes information like the new name of the feature
or an alternative approach to implementing the feature. If the notes are blank, then the name of
the old feature and the new feature are nearly identical (the APIs are likely to be different).

### Directives
Most directives changed to components. These make for straight forward conversions, for instance
`<md-card>` changed to `<mat-card>`. Some element directives changed to attribute directives, for
instance `<md-subheader>` changed to `matSubheader`.

Here is a reference table where you can review and compare the AngularJS Material and
Angular Material documentation. AngularJS Material uses the `md-` and `$md` prefixes while Angular
Material uses the `mat-`, `mat`, and `Mat` prefixes. The `md-` prefixes have been omitted to make
this table more readable.

| Directive          | Old Docs     | New Docs     | Notes                                                  |
|--------------------|--------------|--------------|--------------------------------------------------------|
| autocomplete       |   [Docs][41] |   [Docs][24] |                                                        |
| autofocus          |   [Docs][69] |   N/A        |                                                        |
| button             |   [Docs][43] |   [Docs][1]  |                                                        |
| calendar           |   [Docs][70] |   N/A        |                                                        |
| card               |   [Docs][44] |   [Docs][2]  |                                                        |
| checkbox           |   [Docs][45] |   [Docs][3]  |                                                        |
| chip               |   [Docs][71] |   [Docs][26] |                                                        |
| chip-remove        |   [Docs][72] |   [Docs][26] | `matChipRemove`                                        |
| chips              |   [Docs][46] |   [Docs][26] |                                                        |
| colors             |   [Docs][73] |   N/A        | [`mat-color` mixin][74] supports static theme color lookups. No support for dynamic theming.|
| contact-chips      |   [Docs][75] |   N/A        |                                                        |
| content            |   [Docs][76] |   [Docs][77] | `cdkScrollable`                                        |
| datepicker         |   [Docs][47] |   [Docs][25] |                                                        |
| divider            |   [Docs][49] |   [Docs][35] |                                                        |
| fab-speed-dial     |   [Docs][78] |   N/A        |                                                        |
| fab-toolbar        |   [Docs][79] |   N/A        |                                                        |
| grid-list          |   [Docs][50] |   [Docs][9]  |                                                        |
| highlight-text     |   [Docs][80] |   N/A        |                                                        |
| icon               |   [Docs][51] |   [Docs][10] |                                                        |
| ink-ripple         |   [Docs][81] |   [Docs][19] | `matRipple`                                            |
| input              |   [Docs][52] |   [Docs][5]  | `matInput`                                             |
| input-container    |   [Docs][82] |   [Docs][83] | `mat-form-field`                                       |
| list               |   [Docs][53] |   [Docs][8]  |                                                        |
| menu               |   [Docs][54] |   [Docs][17] |                                                        |
| menu-bar           |   [Docs][84] |   N/A        |                                                        |
| nav-bar            |   [Docs][85] |   [Docs][86] | `mat-tab-nav-bar`                                      |
| nav-item           |   [Docs][87] |   [Docs][88] | `mat-tab-link`                                         |
| optgroup           |   [Docs][89] |   [Docs][90] |                                                        |
| option             |   [Docs][91] |   [Docs][23] |                                                        |
| progress-linear    |   [Docs][55] |   [Docs][12] | `mat-progress-bar`                                    |
| progress-circular  |   [Docs][56] |   [Docs][11] | `mat-progress-spinner`                                 |
| radio-button       |   [Docs][57] |   [Docs][4]  |                                                        |
| radio-group        |   [Docs][92] |   [Docs][93] |                                                        |
| select             |   [Docs][59] |   [Docs][23] |                                                        |
| select-on-focus    |   [Docs][94] |   N/A        |                                                        |
| sidenav            |   [Docs][60] |   [Docs][6]  |                                                        |
| sidenav-focus      |   [Docs][95] |   N/A        | [autofocus][96] only supports focusing the first focusable element.|
| switch             |   [Docs][61] |   [Docs][14] | `mat-slide-toggle`                                     |
| slider             |   [Docs][62] |   [Docs][16] |                                                        |
| slider-container   |   [Docs][97] |   N/A        |                                                        |
| subheader          |   [Docs][98] |   [Docs][99] | `matSubheader`                                         |
| swipe              |   [Docs][100]|   N/A        | See [HammerJS setup][101] and [Hammer.Swipe][102]      |
| tabs               |   [Docs][64] |   [Docs][13] |                                                        |
| textarea           |   [Docs][52] |   [Docs][5]  |                                                        |
| toolbar            |   [Docs][66] |   [Docs][7]  |                                                        |
| tooltip            |   [Docs][67] |   [Docs][18] |                                                        |
| truncate           |   [Docs][103]|   N/A        |                                                        |
| virtual-repeat     |   [Docs][68] |   [Docs][40] | `cdk-virtual-scroll-viewport` and `*cdkVirtualFor`     |
| whiteframe         |   [Docs][104]|  [Guide][105]| Based on classes and mixins                            |

### Services

| Service          | Old Docs     | New Docs     | Notes                                                  |
|------------------|--------------|--------------|--------------------------------------------------------|
| $mdAriaProvider  |   [Docs][124]|   N/A        |                                                        |
| $mdBottomSheet   |   [Docs][42] |   [Docs][38] | `MatBottomSheet`                                       |
| $mdColors        |   [Docs][125]|   N/A        | [`mat-color` mixin][74] supports static theme color lookups. No support for dynamic theming.|
| $mdCompiler      |   [Docs][126]|   N/A        |                                                        |
| $mdCompilerProvider| [Docs][127]|   N/A        |                                                        |
| $mdDateLocaleProvider|[Docs][128]|  [Docs][129]| `MomentDateAdapter`                                    |
| $mdDialog        |   [Docs][48] |   [Docs][22] | `MatDialog`                                            |
| $mdGestureProvider|  [Docs][130]|   N/A        |                                                        |
| $mdIcon          |   [Docs][131]|   [Docs][132]| `svgIcon`                                              |
| $mdIconProvider  |   [Docs][133]|   [Docs][134]| `MatIconRegistry`                                      |
| $mdInkRipple     |   [Docs][58] |   [Docs][19] | `matRippleTrigger`                                     |
| $mdInkRippleProvider|[Docs][135]|   [Docs][136]| `MAT_RIPPLE_GLOBAL_OPTIONS`                            |
| $mdInteraction   |   [Docs][137]|   N/A        |                                                        |
| $mdLiveAnnouncer |   [Docs][111]|   [Docs][112]| CDK `LiveAnnouncer`                                    |
| $mdMedia         |   [Docs][137]|   [Docs][138]| CDK `BreakpointObserver.isMatched()` or Angular Layout's [MediaObserver Service][119]|
| $mdPanel         |   [Docs][109]|   [Docs][110]| CDK `Overlay`                                          |
| $mdPanelProvider |   [Docs][139]|   N/A        |                                                        |
| $mdProgressCircularProvider|[Docs][140]|N/A    |                                                        |
| $mdSidenav       |   [Docs][141]|   N/A        |                                                        |
| $mdSticky        |   [Docs][142]|   N/A        |                                                        |
| $mdTheming       |   [Docs][143]|   N/A        | [Sass mixins][145] support static theming. No support for dynamic theming.|
| $mdThemingProvider|  [Docs][144]|   N/A        | [Sass mixins][146] support static, custom theme creation. Use Angular's [Meta Service][147] for browser coloring features.|
| $mdToast         |   [Docs][63] |   [Docs][21] | `MatSnackBar`                                          |

### <a name="new-components"></a> New Components
These are new components found in Angular Material and the Angular CDK that do not exist in
AngularJS Material.

The `mat-` and `cdk-` prefixes have been omitted to make this table more readable.

| Directive        |     Docs     | Notes                                                  |
|------------------|--------------|--------------------------------------------------------|
| badge            |   [Docs][37] |                                                        |
| button-toggle    |   [Docs][15] |                                                        |
| table            |   [Docs][28] | Non-themed CDK and Material versions available.        |
| drag-drop        |   [Docs][39] | CDK                                                    |
| expansion-panel  |   [Docs][32] |                                                        |
| paginator        |   [Docs][29] |                                                        |
| sort-header      |   [Docs][30] |                                                        |
| stepper          |   [Docs][33] | Non-themed CDK and Material versions available.        |
| tree             |   [Docs][36] | Non-themed CDK and Material versions available.        |

 [0]: https://github.com/angular/flex-layout/wiki
 [1]: https://material.angular.io/components/button/overview
 [2]: https://material.angular.io/components/card/overview
 [3]: https://material.angular.io/components/checkbox/overview
 [4]: https://material.angular.io/components/radio/overview
 [5]: https://material.angular.io/components/input/overview
 [6]: https://material.angular.io/components/sidenav/overview
 [7]: https://material.angular.io/components/toolbar/overview
 [8]: https://material.angular.io/components/list/overview
 [9]: https://material.angular.io/components/grid-list/overview
[10]: https://material.angular.io/components/icon/overview
[11]: https://material.angular.io/components/progress-spinner/overview
[12]: https://material.angular.io/components/progress-bar/overview
[13]: https://material.angular.io/components/tabs/overview
[14]: https://material.angular.io/components/slide-toggle/overview
[15]: https://material.angular.io/components/button-toggle/overview
[16]: https://material.angular.io/components/slider/overview
[17]: https://material.angular.io/components/menu/overview
[18]: https://material.angular.io/components/tooltip/overview
[19]: https://material.angular.io/components/ripple/overview
[20]: https://material.angular.io/guide/theming
[21]: https://material.angular.io/components/snack-bar/overview
[22]: https://material.angular.io/components/dialog/overview
[23]: https://material.angular.io/components/select/overview
[24]: https://material.angular.io/components/autocomplete/overview
[25]: https://material.angular.io/components/datepicker/overview
[26]: https://material.angular.io/components/chips/overview
[27]: https://material.angular.io/guide/typography
[28]: https://material.angular.io/components/table/overview
[29]: https://material.angular.io/components/paginator/overview
[30]: https://material.angular.io/components/sort/overview
[31]: https://tina-material-tree.firebaseapp.com/simple-tree
[32]: https://material.angular.io/components/expansion/overview
[33]: https://material.angular.io/components/stepper/overview
[34]: https://material.angular.io/cdk/categories
[35]: https://material.angular.io/components/divider/overview
[36]: https://material.angular.io/components/tree/overview
[37]: https://material.angular.io/components/badge/overview
[38]: https://material.angular.io/components/bottom-sheet/overview
[39]: https://material.angular.io/cdk/drag-drop/overview
[40]: https://material.angular.io/cdk/scrolling/overview#virtual-scrolling
[41]: https://material.angularjs.org/latest/api/directive/mdAutocomplete
[42]: https://material.angularjs.org/latest/api/service/$mdBottomSheet
[43]: https://material.angularjs.org/latest/api/directive/mdButton
[44]: https://material.angularjs.org/latest/api/directive/mdCard
[45]: https://material.angularjs.org/latest/api/directive/mdCheckbox
[46]: https://material.angularjs.org/latest/api/directive/mdChips
[47]: https://material.angularjs.org/latest/api/directive/mdDatepicker
[48]: https://material.angularjs.org/latest/api/service/$mdDialog
[49]: https://material.angularjs.org/latest/api/directive/mdDivider
[50]: https://material.angularjs.org/latest/api/directive/mdGridList
[51]: https://material.angularjs.org/latest/api/directive/mdIcon
[52]: https://material.angularjs.org/latest/api/directive/mdInput
[53]: https://material.angularjs.org/latest/api/directive/mdList
[54]: https://material.angularjs.org/latest/api/directive/mdMenu
[55]: https://material.angularjs.org/latest/api/directive/mdProgressLinear
[56]: https://material.angularjs.org/latest/api/directive/mdProgressCircular
[57]: https://material.angularjs.org/latest/api/directive/mdRadioButton
[58]: https://material.angularjs.org/latest/api/service/$mdInkRipple
[59]: https://material.angularjs.org/latest/api/directive/mdSelect
[60]: https://material.angularjs.org/latest/api/directive/mdSidenav
[61]: https://material.angularjs.org/latest/api/directive/mdSwitch
[62]: https://material.angularjs.org/latest/api/directive/mdSlider
[63]: https://material.angularjs.org/latest/api/service/$mdToast
[64]: https://material.angularjs.org/latest/api/directive/mdTabs
[65]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout
[66]: https://material.angularjs.org/latest/api/directive/mdToolbar
[67]: https://material.angularjs.org/latest/api/directive/mdTooltip
[68]: https://material.angularjs.org/latest/api/directive/mdVirtualRepeat
[69]: https://material.angularjs.org/latest/api/directive/mdAutofocus
[70]: https://material.angularjs.org/latest/api/directive/mdCalendar
[71]: https://material.angularjs.org/latest/api/directive/mdChip
[72]: https://material.angularjs.org/latest/api/directive/mdChipRemove
[73]: https://material.angularjs.org/latest/api/directive/mdColors
[74]: https://material.angular.io/guide/theming-your-components#note-using-the-code-mat-color-code-function-to-extract-colors-from-a-palette
[75]: https://material.angularjs.org/latest/api/directive/mdContactChips
[76]: https://material.angularjs.org/latest/api/directive/mdContent
[77]: https://material.angular.io/cdk/scrolling/overview#cdkscrollable-and-scrolldispatcher
[78]: https://material.angularjs.org/latest/api/directive/mdFabSpeedDial
[79]: https://material.angularjs.org/latest/api/directive/mdFabToolbar
[80]: https://material.angularjs.org/latest/api/directive/mdHighlightText
[81]: https://material.angularjs.org/latest/api/directive/mdInkRipple
[82]: https://material.angularjs.org/latest/api/directive/mdInputContainer
[83]: https://material.angular.io/components/form-field/overview
[84]: https://material.angularjs.org/latest/api/directive/mdMenuBar
[85]: https://material.angularjs.org/latest/api/directive/mdNavBar
[86]: https://material.angular.io/components/tabs/overview#tabs-and-navigation
[87]: https://material.angularjs.org/latest/api/directive/mdNavItem
[88]: https://material.angular.io/components/tabs/api#MatTabLink
[89]: https://material.angularjs.org/latest/api/directive/mdOptgroup
[90]: https://material.angular.io/components/select/overview#creating-groups-of-options
[91]: https://material.angularjs.org/latest/api/directive/mdOption
[92]: https://material.angularjs.org/latest/api/directive/mdRadioGroup
[93]: https://material.angular.io/components/radio/overview#radio-groups
[94]: https://material.angularjs.org/latest/api/directive/mdSelectOnFocus
[95]: https://material.angularjs.org/latest/api/directive/mdSidenavFocus
[96]: https://material.angular.io/components/sidenav/api#MatSidenav
[97]: https://material.angularjs.org/latest/api/directive/mdSliderContainer
[98]: https://material.angularjs.org/latest/api/directive/mdSubheader
[99]: https://material.angular.io/components/list/overview#lists-with-multiple-sections
[100]: https://material.angularjs.org/latest/api/directive/mdSwipeDown
[101]: https://material.angular.io/guide/getting-started#step-5-gesture-support
[102]: http://hammerjs.github.io/recognizer-swipe/
[103]: https://material.angularjs.org/latest/api/directive/mdTruncate
[104]: https://material.angularjs.org/latest/api/directive/mdWhiteframe
[105]: https://material.angular.io/guide/elevation
[106]: https://material.angularjs.org/latest/Theming/01_introduction
[107]: https://material.angularjs.org/latest/CSS/typography
[108]: https://material.angularjs.org/latest/layout/introduction
[109]: https://material.angularjs.org/latest/api/service/$mdPanel
[110]: https://material.angular.io/cdk/overlay/overview
[111]: https://github.com/angular/material/blob/v1.1.17/src/core/services/liveAnnouncer/live-announcer.js
[112]: https://material.angular.io/cdk/a11y/api#LiveAnnouncer
[113]: https://github.com/angular/material/blob/v1.1.17/src/core/util/util.js#L86-L105
[114]: https://material.angular.io/cdk/bidi/overview
[115]: https://material.angular.io/cdk/text-field/api#CdkTextareaAutosize
[116]: https://github.com/angular/flex-layout/wiki/Declarative-API-Overview
[117]: https://github.com/angular/flex-layout/wiki/Responsive-API
[118]: https://github.com/angular/flex-layout/wiki/Breakpoints
[119]: https://github.com/angular/flex-layout/wiki/API-Documentation#javascript-api-imperative
[120]: https://github.com/angular/flex-layout
[121]: https://material.angular.io/cdk/layout/overview
[122]: https://material.io/design/layout/responsive-layout-grid.html#breakpoints
[123]: https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList
[124]: https://material.angularjs.org/latest/api/service/$mdAriaProvider
[125]: https://material.angularjs.org/latest/api/service/$mdColors
[126]: https://material.angularjs.org/latest/api/service/$mdCompiler
[127]: https://material.angularjs.org/latest/api/service/$mdCompilerProvider
[128]: https://material.angularjs.org/latest/api/service/$mdDateLocaleProvider
[129]: https://material.angular.io/components/datepicker/overview#internationalization
[130]: https://material.angularjs.org/latest/api/service/$mdGestureProvider
[131]: https://material.angularjs.org/latest/api/service/$mdIcon
[132]: https://material.angular.io/components/icon/api#MatIcon
[133]: https://material.angularjs.org/latest/api/service/$mdIconProvider
[134]: https://material.angular.io/components/icon/api#MatIconRegistry
[135]: https://material.angularjs.org/latest/api/service/$mdInkRippleProvider
[136]: https://material.angular.io/components/ripple/overview#global-options
[137]: https://material.angularjs.org/latest/api/service/$mdMedia
[138]: https://material.angular.io/cdk/layout/overview#breakpointobserver
[139]: https://material.angularjs.org/latest/api/service/$mdPanelProvider
[140]: https://material.angularjs.org/latest/api/service/$mdProgressCircular
[141]: https://material.angularjs.org/latest/api/service/$mdSidenav
[142]: https://material.angularjs.org/latest/api/service/$mdSticky
[143]: https://material.angularjs.org/latest/api/service/$mdTheming
[144]: https://material.angularjs.org/latest/api/service/$mdThemingProvider
[145]: https://material.angular.io/guide/theming-your-components
[146]: https://material.angular.io/guide/theming#defining-a-custom-theme
[147]: https://angular.io/api/platform-browser/Meta
[148]: https://material.io/archive/guidelines/style/color.html#color-color-palette
[149]: https://material.angularjs.org/latest/Theming/06_browser_color
[150]: https://material.angularjs.org/latest/Theming/01_introduction#palettes
[151]: https://material.angularjs.org/latest/Theming/03_configuring_a_theme#defining-custom-palettes
[152]: https://material.angularjs.org/latest/Theming/03_configuring_a_theme
[153]: https://material.angularjs.org/latest/Theming/04_multiple_themes
[154]: https://material.angularjs.org/latest/Theming/04_multiple_themes#using-another-theme
[155]: https://material.angularjs.org/latest/Theming/04_multiple_themes#via-a-directive
[156]: https://material.angularjs.org/latest/Theming/04_multiple_themes#dynamic-themes
[157]: https://material.angularjs.org/latest/Theming/05_under_the_hood
[158]: https://developer.android.com/guide/topics/ui/look-and-feel/themes
[159]: https://msdn.microsoft.com/en-us/library/windows/apps/mt210949.aspx
[160]: https://caniuse.com/#feat=css-variables
[161]: https://github.com/angular/material2/releases/tag/7.3.3
[162]: https://github.com/angular/material2/issues/4352
[163]: https://material.angular.io/guide/theming#using-a-pre-built-theme
[164]: https://material.angular.io/guide/theming#defining-a-custom-theme
[165]: https://material.angular.io/guide/theming#multiple-themes
[166]: https://material.angular.io/guide/theming#theming-only-certain-components
[167]: https://github.com/angular/material2/blob/7.3.6/src/lib/core/theming/_palette.scss#L72-L103
[168]: https://material.angular.io/guide/theming-your-components
[169]: https://material.angularjs.org/latest/Theming/03_configuring_a_theme#specifying-custom-hues-for-color-intentions
[170]: https://sass-lang.com/
[171]: https://material.angular.io/
[172]: https://blog.angular.io/stable-angularjs-and-long-term-support-7e077635ee9c
[173]: https://angular.io/guide/upgrade#preparation
[174]: https://material.angular.io/guide/typography#customization
[175]: https://material.angular.io/guide/typography#material-typography-in-your-custom-css

[aio]: https://angular.io/
[cdk]: https://material.angular.io/cdk
[ngUpgrade]: https://angular.io/guide/upgrade-performance
[style-guide]: https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md
[webpack]: http://webpack.github.io/
