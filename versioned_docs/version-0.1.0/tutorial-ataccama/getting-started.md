---
sidebar_position: 2
---

# Getting started

## Check the app in our staging environment

Look how it looks like at [https://dev20.aks.ondemand.ataccama.dev/](https://dev20.aks.ondemand.ataccama.dev/) (login `admin:admin`)

## Let's run the application locally

[Set up the FE development](https://www.notion.so/Set-up-the-FE-development-80b8fd310231468a8a1bc439026b194f)

[Running BE locally](https://www.notion.so/Running-BE-locally-480860b09d524d88bf255d7a6dc2893a)

## Spin up your own environment

Read about how to spin up your very own environment in our company infrastructure.

[FE - How to deploy using DevSpace](https://www.notion.so/FE-How-to-deploy-using-DevSpace-283f970d95f44a92b6f18a5d18249725)

## Go through the tutorial

Let's take a tour around the codebase and focus on the key concepts of our app.

We have created a tutorial plugin for you to explore. Open the one-metadata-frontend project, open `src/plugins/index.ts` in the project root and uncomment the line with tutorial plugin.

```jsx
// tutorial: () => import(/* webpackChunkName: "plugins/tutorial-fe/index" */ './tutorial'),
```

Then run the app and look for a new tutorial navigation item in the navigation bar.

[Tutorial plugin](https://www.notion.so/Tutorial-plugin-180e1810bf174671adcec34237f88860)

## How to navigate in the code

Due to the nature of the app and the code structure, it may be difficult at first to find your way around. Here are few tips that can help you

## Ataccama dev tools

One 2.0 has its own dev tools which can be a huge help. Open the console in the browser dev tools and write `openDebuggingTools()` to open them.

<aside>
💡 You can also use shortcut `Cmd+Option+T` on Mac or `ctrl+alt+t` on Win to open the debug tools.

</aside>

In the Ata dev tools, you can see few tabs. The most relevant are

- **Screen entity** - this shows what entity and entity data you have available on the screen (shows only keys and value types, but not values)
    <aside>
    💡 Aside from normal types like `string` or `boolean` you can see also some shortcuts: S, SRE, SEE, AEE. To see what they means go check [Entity fetch rules](https://www.notion.so/Entity-fetch-rules-182c09824e6f4964a0888cc2e8b9c3d1)
    
    </aside>

- **Screen layout** - this tab will show you JSON layout of the screen. So you can see what widgets are rendered and easily found this in the codebase. This can be also useful if you need to create a new custom layout to get the basic structure by copying this.
- **Fetch rules** - here you can see add new fetch rules to get the additional data if needed

Screen layout and Fetch rules tabs are editable. You can modify the layout, add new widgets and fetch rules and experiment with it. Once you want to see your changes in the app hit `Cmd+S` to save the changes. All the changes are persisted even after page refresh and killing FE dev server.

In case you need to get rid of those changes, type some non-existing widget code into any `_type` property in the screen layout. You will see the error page and button to reset the layout to its original state.

<aside>
💡 When you play with the screen layout and then decide to move to the real code. Don't forget to reset the layout. Otherwise, you will not see your changes. It is easy to forget this sometimes and lost some time trying to figure out why your isn't code working 😉.

</aside>

## Other navigation tips

- Unless you work on the core, most of the logic is in the `plugins` directory. Good point where to start looking is the plugin which handles section you are working on. Eg. lookup items are in `rdm` plugin, Catalog items in `catalog` plugin (naming is quite good and you should be able to identify the plugin name usually by looking at URL).
  Then it may be worth checking `config` folder for `ScreenConfig.overrides.ts` or `EntityConfig.overrides.ts` to see what layouts or widgets are used for specific screens or entities (Ata dev tools can help with this as well).
- In JSON layouts you will see `_type` property. Its value (eg. `rdm.sourceItem.editor`) is mapped to real components in the `<PLUGIN_FOLDER>/widgets/index.ts` file. So when locking for a specific components based on their alias in the layout files.
    <aside>
    💡 Naming convention is that first part of the widget name is the plugin name. So `rdm.sourceItem.editor` widget is in `rdm` plugin folder.
    
    </aside>

- Often you can see that the entity properties are rendered by `PropertyRest.tsx` widget. But that doesn't tell you what exact widget is used for the property itself. You can find this out in the plugin's `EntityPluginConfig.overrides.ts`. Look for the name of your property and what widget is referenced in `labelWidget` config option (for detail view, for edit/create look for `editorWidget` and for listing look for `cellWidget`).
- Sometime properties are rendered by default renderers, not special widgets defined in the plugin folder. To see default layouts for the entities (S, SRE etc.) Go to `plugins/entity/config/EntityPluginConfig.defaults.ts`. You can see all the default widgets used in there.
