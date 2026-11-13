---
title: Advanced Configuration
Description: Documentation related to advanced configuration of the dashboard
---

# Advanced Configuration

## Dashboard Parameters

A dashboard may have one or more parameters to modify its content (data and titles).
Any parameter will be displayed in the **dashboard parameters panel,** where the end-user can select or type values and click **Apply** to update the dashboard with new parameter values.

The example below displays a simple HR statistics dashboard for a given organization within the enterprise (count of personnel in the organization, and jobs breakdown).
The end-user can then select an organization from a list of known organizations to update the dashboard with data about the selected organization.

![Organization HR statistics](./images/params_01.png "Organization HR statistics")

A dashboard can have two kinds of parameters:

- **User editable parameters**: These are the regular parameters. The value of these parameters can be changed by the end-user. These parameters can affect dashboard component data and titles.
- **Hidden parameters**: The value of these parameters must be set at design time only. The parameter value can also be part of the dashboard title. Hidden parameters are mostly useful with **Dashboard Templates** (see section below).

### User Editable Parameters

**User editable parameters** are the common type of dashboard parameters.
They are defined by the designer, and their values can be changed by the end-user through the dashboard parameter panel.

The value of each parameter can be used to modify dashboard component data and/or titles.
The parameter value can either be typed in a text box or selected from a list of possible values by the end-user.

The possible values can either be dynamic and come from a view, or be a static list of values.
The designer can also define a default text or item to be used when the dashboard is displayed for the first time.

### Hidden Parameters

The common purpose of **hidden parameters** is to allow creating separate instances of a **dashboard template** (read more on dashboard templates in the section below).
These dashboard instances can then be published to different populations.

For example, consider there are many account repository (domain forest) managers in your company, who all need a similar dashboard with analytics for the accounts in the repository they manage.
In this case, the designer would create a dashboard template that displays account analytics for a given repository, as a hidden parameter.
He will then create instances of the dashboard template for each managed repository, and publish them to each repository manager.
For convenience, the dashboard title could automatically include the repository name, _e.g._ "Access Analytics on EMEA".

Hidden parameters have the same behavior as regular parameters, with the following differences:

- A hidden parameter does not show in the dashboard parameter panel. Its value must be set at design time, usually when instantiating the dashboard from a template.
- The hidden parameter value can affect dashboard component data and titles, plus it can also be used in the dashboard's title itself.

### Create a Dashboard Parameter

Dashboard parameters are managed via the **Parameters** tab of the dashboard's configuration wizard.

![Managing dashboard parameters](./images/params_02.png "Managing dashboard parameters")

The designer can do the following:

- Create a new parameter
- Modify an existing parameter
- Delete a parameter
- Move parameters up and down to change their display order in the dashboard parameter panel (from left to right)

To create a new parameter:

- Click **Create New**... to display the parameter creation dialog box

![Creating a parameter](./images/dashparam0.png "Creating a parameter")

- Type in **Name** a unique name for the parameter. This name is used to identify the parameter in component titles.  
  Use a meaningful but short name, such as **org**. Don't use anonymous names such as param1. Parameter names should not contain spaces.
- Type in **Label** a label for the parameter. This label will be displayed as the title for the parameter in the dashboard's parameter panel.
- Optionally, type in **Help text** a text describing the purpose of the parameter. This text will show as a tooltip for the parameter in the dashboard's parameter panel.
- Check **Editable by end-user** for regular user editable parameters, and uncheck it for hidden parameters
- Select a user interface type for editing the parameter:
  - **Text box**: the parameter value will be typed in a text box
    - Optionally, type in **Default Value** a default value for the parameter.
  - **Menu**: the parameter value can be selected from a list.
- For Menu-based parameters, select the type of values that will be displayed in the menu:
  - **Dynamic values** to display dynamic values coming from a view
  - **Static values** to display a static list of values

For **Dynamic values** menus:

- Select the **Source View** that will provide items for the parameter menu.  
  The view must have the "**Allow selection of the view in the portal**" option to be listed as a source view.
- If the view has parameters, click **Params...** to edit the parameters of the view.
- Select in **Value column** a column from the selected view that will provide the value of the parameter.
  This value will usually be used to modify data in the dashboard. It could be, for example, an organization permanent UID or code.
- Select in **Label column** a column from the selected view that will provide the label of the parameter value to be displayed in the parameter panel.
  The parameter label can also be included in component titles.
- Check the **Allow empty value** option to allow a null value to be selected for this parameter.
  You can type an optional label for this empty value, for example "_All Organisations_". The actual meaning and behavior for this null value is up to you.
  If this option is not checked, the first value in the list is automatically selected in the displayed mashup dashboard parameter panel.
- Optionally, select a **Default value** for the parameter, to be used when the dashboard is displayed for the first time.

![Dynamic values](./images/dash_param1.png "Dynamic values")

> It's recommended to use dedicated views to provide values for parameter menus, with only the columns that are used for values and labels.
> This is to ensure optimal performance when displaying the dashboard parameter panel with large datasets (_e.g._ listing accounts).  
> Don't use general purpose views with dozens of columns.
>
> Whenever possible, you should use permanent, timeslot-independent, and environment-independent key attributes as value columns for parameters, to ensure maximum portability of the dashboard across timeslots and environments.
> This is especially important for hidden parameters and for parameter default values, because the value is stored with the dashboard and must be valid across timeslots and across environments (UAT, Production).

For **static values** menus:

- Click **Create** to add a new static value to the list. You must then provide a **Label** and a **Value** for each item in the list.
  - The label will be displayed in the UI dashboard parameter panel and in component titles.
  - The value will be passed as parameters to dashboard data.
- You can **Create**, **Edit**, **Delete** or **Delete All** menu items
- Optionally, select a **Default value** for the parameter, to be used when the dashboard is displayed for the first time.

![Static values](./images/params_04.png "Static values")

### Use Parameters in a Dashboard

Parameter values can be used to modify dashboard component data and titles and, only for hidden parameters, the dashboard title itself.

To modify a dashboard component's data based on the value of a parameter, do the following:

- Open the component's **Configuration** wizard
- Select the **Search** or **Value** tab of the component's configuration
- In simple free search mode:
  - define the search criteria
  - when selecting a value for a given criterion, check the **Dashboard parameter** option; the value menu will now display the list of dashboard parameters
    - select the dashboard parameter to use
      ![ Using parameters in dashboard search ](./images/dashboard_search_param.png "Using parameters in dashboard search")
- In advanced rule mode:
  - Make sure that the search **rule** has at least one parameter
  - Click the **Parameters** button to configure the rule parameters
  - For the rule parameter that should use the dashboard parameter, select **Dashboard Parameter** as the source
  - Select the dashboard parameter to use from the parameter list (the parameter's label and name are displayed in the list)

![Dashboard Parameter](./images/dash_param6.png "Dashboard Parameter")

You can also use dashboard parameters in a table or chart component to filter secondary attributes.

To include a dashboard parameter's label in a dashboard component's title, do the following:

- Open the component's **Configuration** wizard
- Select the **Basic** tab of the component's configuration
- Click in the **Title** text box, and move the insertion bar to where the parameter label should be placed inside the title
- Select a parameter from the **Insert Variable** list (both user-editable and hidden parameters are listed); the parameter will be inserted as a **{param.xxxx}** token.
- You could also directly type the parameter token in the title box

![Dashboard Parameter 2](./images/params_06.png "Dashboard Parameter 2")

To include a hidden parameter label in a dashboard's title, do the following:

- Open the dashboard's **Configuration** wizard
- Select the **General** tab
- Click in the **Title** text box, and move the insertion bar to where the hidden parameter label should be placed inside the title
- Select a parameter from the **Insert Variable** list (only hidden parameters are listed); the parameter will be inserted in the title as a **{param.xxxx}** token.
- The actual value of the hidden parameter can be set in the **Instance** tab (see § Dashboard Template section)
- Click **Finish** to validate your configuration
- Notice how the dashboard title (and the corresponding menu entry) has changed.

![Dashboard Parameter 3](./images/params_07.png "Dashboard Parameter 3")

### Cascading Parameters

New in **Curie R1**

Cascading parameters are useful when selecting a value from items that are hierarchically organized, for example `Application > Permission`, `Repository > Account`, etc.
Cascading parameters mean that the list of values proposed for a parameter depends on the selected value of another parameter.

To define cascading parameters, do the following:

- Define the parent parameter, for example `app (Applications)`
- Define the child parameter, for example `perm (Permission)`
  - select **Dynamic values** for the source
  - select the **Source View**, which must accept a parameter that points to the parent parameter selection value (_e.g._ application permanent identifier).
    If the view does not exist, create it and refresh the list of views using the refresh icon.
  - click the **Params** button, select the parameter of the view and set its source to **Dashboard param.**
  - select the parent parameter (in this case, `Application {param.app}`) from the value list.

![Cascading parameter  ](./images/cascading_param_cfg.png "Cascading parameter")

This is what the dashboard parameter panel will look like:

![Cascading parameter  ](./images/dashboard_cascading_param.png "Cascading parameter")

## Dashboard Templates

The main purpose of **Dashboard Templates** is to streamline the creation of similar dashboards that differ only in their scopes.

Consider the above example, where there are many account repositories, one per area, each being managed by a different team. The managers all need a similar dashboard with analytics for the accounts in the repository they manage. The scope of the dashboard is the repository.

In this case, a designer would create a **dashboard template** that displays access analytics for a given repository, as a hidden parameter.
He will then **instantiate** the dashboard template for each of the managed repositories, and publish them to each repository manager.
For convenience, the dashboard title could automatically include the repository name, _e.g._ "Access Analytics on EMEA".

To create a dashboard template or to turn a dashboard into a template, do the following:

- Open the dashboard's configuration wizard
- In the **General** tab, check the **Template** option
- In the **Parameters** tab, create one or more hidden parameters, with the option "**Editable by end-users**" unchecked, which will serve as the dashboard's scope parameters.
- In the **General** tab again, you can optionally include the parameters' labels in the dashboard title (see section **Hidden Parameters** above)

To create an instance of a dashboard template, do the following:

- In the **Dashboard Manager** page,
- Select the dashboard template to instantiate,
- Click the **Instantiate...** button; this opens the dashboard instantiation wizard
- In the **Instance** tab, select or type a value for each of the dashboard's hidden parameters.
- Click **Finish** to create the dashboard instance
- The new dashboard instance appears in the list of dashboards
- The dashboard can be published as is to its target audience, or modified before publication
- The dashboard instance parameters can be modified at any time, through the **Instance** tab of the dashboard configuration.
- You can create as many dashboard instances of the same template as needed.

![Dashboard Manager ](./images/template_1.png "Dashboard Manager ")

The dashboard instances can then be published as is to their respective audiences, or be modified before publication.
After creation, the new instance dashboards are independent of their template, and further changes to the template (or even its deletion) won't be reflected in the dashboard instances.

## Secondary Parameter Configuration

**Secondary Parameters** may be involved in different places in a dashboard's configuration:

- **Component Rule with parameters**: Dashboard component (table, chart, gauge) Search Rule that may have parameters,
- **Component View with parameters**: Dashboard component (table, chart) View that may have parameters,
- **Dashboard Parameter Menu View with parameters**: Dashboard parameter possible values coming from a View that may have parameters,
- **Component Link Target with parameters**: Dashboard component (chart, table or gauge) link to another page or dashboard that may have parameters,

In all these cases, you can define for each involved secondary parameter where its actual value will come from (secondary parameter **Source**), and what the actual value will be (secondary parameter **Value**).

There are four secondary parameter source types:

- **Value**: the secondary parameter value is a predefined value that must be set at design-time (typed or selected from a list of possible values)
- **Dashboard Parameter**: the secondary parameter value will be taken from one of the dashboard's main parameters' runtime values
- **User**: the secondary parameter will take its value from one of the currently connected user's attributes
- **Clicked Item**: the secondary parameter will take its value from the chart item or table row that was just clicked

Some source types only make sense for some secondary parameter kinds, _e.g._ Clicked Item for Table and Chart Link target parameters.
The following table shows which source types apply to which secondary parameter kinds:

|                                                  | **Fixed Value**         | **Dashboard Parameter** | **User Attribute** | **Clicked Item Attribute** |
| ------------------------------------------------ | ----------------------- | ----------------------- | ------------------ | -------------------------- |
| **Component Rule w/ params**                     | X                       | X                       | X                  |                            |
| **Component View w/ params**                     | X                       | X                       | X                  |                            |
| **Dashboard Parameter Menu <br> View w/ params** | X                       |                         | X                  |                            |
| **Gauge Link Target w/ params**                  | X                       | X                       | X                  |                            |
| **Table and Chart Link Target <br> w/ params**   | X                       | X                       | X                  | X                          |

### Fixed Value Source

The secondary parameter's actual value is a fixed value that must be set at design-time.
Fixed values can be used in all situations.
This could be helpful to pass ad-hoc static values to rules, views, links, etc.

To use a fixed value as a source type, do the following:

- Select **Value** as the **Source** in the **Parameters** configuration panel
- Either type the fixed value in the text box or select the value from the list of proposed values (the UI is automatically determined based on the type of the secondary parameter)

![Fixed Value Source ](./images/params2_01.png "Fixed Value Source")

### Dashboard Parameter Source

The secondary parameter value will be passed from the current value of a given dashboard parameter.
When the dashboard's end-user selects or types another value for the related dashboard parameter, and clicks **Apply**, the secondary parameter value is automatically updated.

Dashboard source types can be used with all secondary parameters, except to filter dashboard parameters themselves, so as to avoid recursion.
This source type is the principal means of using dashboard parameters in dashboard content.
Dashboard parameter values can also be passed to other dashboards or pages through links.

To use a dashboard's parameter as a source type, do the following:

- Select **Dashboard Parameter** as the **Source** in the **Parameters** configuration panel, if the choice is available
- Select one of the dashboard's parameters to use as the source. Each dashboard parameter in the list is displayed both with its label, and the syntax **{param.<param_name>}**

![Dashboard Parameter Source](./images/params2_02.png "Dashboard Parameter Source")

### User Attributes Source

The secondary parameter will take its value from characteristics of the currently connected user.
This is especially useful to create dashboards whose content automatically adapts to the current user (without the user having to select a parameter).
This source type can be used with all secondary parameter types.

To use an attribute of the current user as a source type, do the following:

- Select **User** as the **Source** in the **Parameters** configuration panel
- Select one of the user's attributes from the list.

![User Attributes Source ](./images/params2_03.png "User Attributes Source ")

- The proposed attributes are the columns of the **user.principal.view** View, which is configured in the project's **Configuration** > **Web Portal** > **Other Properties** section.  
  The main purpose of this view is to validate the login used to connect to the Identity Analytics portal. It's also used to provide additional user-related attributes.  
  You can design a view of your own to validate the login and provide custom values to use in secondary parameters (restart the server to make them effective).

![Other Properties](./images/params2_04.png "Other Properties ")

#### Simulate Another User

During dashboard design, the actual values for user attributes used in secondary parameters will be those of the designer's account (which is the currently connected user at design time).
Fortunately, you can simulate another user account while in design mode to test your dashboard. To do so:

- Select **Dashboard User Simulation...** from the dashboard's menu, to open the **Dashboard user simulation** dialog box

![Simulating another user](./images/params2_05.png "Simulating another user")

- Check **Simulate dashboard login** and type a valid login to simulate another user
- or uncheck **Simulate dashboard login** to go back to using the designer's account
- Click **OK** to validate the simulated account against the configured **user.principal.view**
- When changing to another account, the dashboard parameter lists and content are updated to reflect the new user attributes.  
  If not, hit F5 to refresh the dashboard and take into account the new user characteristics.
- The simulation will be effective during the designer's user session or until manually cleared. It will affect all dashboards that have secondary user parameters.  
  The dashboard title indicates whether a user simulation is active.

![Manager dashboard](./images/params2_06.png "Manager dashboard")

- User simulation neither affects native _View perimeter_ results nor regular pages, which are based on the actual connected user account.

### Clicked Item Attributes Source

The secondary parameter will take its value from the chart item (bar, wedge) or table row that was just clicked, when links are enabled for this widget.
This source type is only available to configure dashboard chart / table widget links and is the main means of passing along context parameters to the link.

To use an attribute of the clicked item as a source for secondary parameters, do the following:

- Select **Clicked Item** as the **Source** in the **Links** configuration page **Parameter** area, for the target page or target dashboard.
- Select one of the clicked item attributes from the **Value** list (which are the columns of the widget's View) to pass to the secondary parameter.

![Attributes Source](./images/params2_07.png "Attributes Source")

## Managing Dashboard Links

**Dashboard Links** allow you to click on any dashboard widget and go to another page, dashboard, detail dialog box or even an external web site, in order to get information or carry out an action on the clicked linked element.

Links are available on all dashboard widgets:

| **Widget** | **Linked item**                | **Link presentation**                                                                                                                           |
| ---------- | ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Table      | any row of the table           | Text of one of the row's columns is displayed as an HTML link (blue underlined text). <br> ![Link table](./images/link_table.png "Link table") |
| Chart      | bar/section/wedge of the chart | The cursor will turn to a hand when moved over the bar item when links are enabled <br> ![Link chart](./images/link_chart.png "Link chart")     |
| Gauge      | The gauge as a whole           | The cursor will turn to a hand when moved over the gauge when links are enabled <br> ![Link gauge](./images/link_gauge.png "Link gauge")        |

Link targets can be of these kinds:

- A detail page (for example, detail on a given account, identity, group, application, etc.),
- Any given page or report page accessible to the end-user, possibly with parameters,
- Another dashboard accessible to the end-user, possibly with parameters,
- A detail dialog box,
- An external web site, possibly passing contextual parameters in the URL. The linked external page will be opened in a separate browser tab.

Contextual information and information on the clicked item can be passed to the link target.

### Link configuration

To define a link on a widget, do the following:

- Check **Enable links** in the configuration wizard's **Links** tab to enable the links for this widget.
- Select from the **Link to** menu the target link type, from these possibilities:
  - **Detail page**: this is the standard detail page for any given Ledger entity (such as identity, account, application, group, etc.)
  - **Another page**: a custom page in the web portal, usually with parameters
  - **Another dashboard:** another dashboard, usually with parameters, that is accessible to the user.
  - **Search detail dialog box**: a dialog box that displays a list of items involved in search-based gauge figure computations.
  - **External URL:** a page in another web site. Parameters are passed in the URL search.
- For **table** widgets, select from **Display link in column** the table's column where the link should be displayed.
  Usually it will be the left-most column, but it could be any column of the table. Only one column can be displayed as a link.
- Additional link configuration will vary depending on the selected target type (see below).
- Make sure to **Save** the dashboard before testing the link and getting back to view mode, because you cannot go to another page while the dashboard is being edited.

#### Detail Page Link

- Choose from the **Selected item UID attribute** menu the view column's value to use as the permanent identifier for the detail page.
  If the list is empty, you will need to add a permanent identifier to the widget's view.  
  The entity type to display (_e.g._ identity, account, group) will be automatically determined from the selected column.

![Detail page link](./images/link_detail.png "Detail page link")

#### Another Page Link

- Select from the **Target page** menu the target page of the link.  
  Each item displays the page's label and that page's tags within brackets. You can use both to filter the page you are targeting.
  Note that all pages with a **<u>tag</u>** are listed in the menu, so you have to make sure that the selected page will actually be accessible to the end user.
- If the page has parameters, you can define the value to use for each of its parameters:
  - **None**: to leave the page parameter empty. This is useful for non-mandatory parameters, if the default value suits your needs.
  - **Value:** to use a static predefined value for the parameter. Type the value to use in the text field.
  - **Clicked Item**: to use an attribute from the clicked row or chart part.  
    You can select any column from the view, even if it is not displayed in the widget.
    Note that this choice is not available for **gauge** links, as there is no clickable sub-item.
  - **Dashboard parameter**: to use a global parameter from the dashboard as a contextual value for the page parameter. Select the dashboard parameter from the list.
  - **User**: to use an attribute related to the currently connected user. Select the attribute from the list.

![Another page link](./images/link_page_cfg.png "Another page link")

#### Another Dashboard Link

- Select from the **Target dashboard** menu the target dashboard of the link. Type to filter by the label of the dashboard.
  Note that all dashboards are listed in the menu, so you have to make sure that the selected dashboard will actually be accessible to the end user.
- If the dashboard has parameters (either editable or hidden), you can define the value to use for each of its parameters:
  - **None**: to leave the dashboard parameter empty. This is useful for non-mandatory parameters, if the default value suits your needs.
  - **Value:** to use a static predefined value for the parameter. Type the value to use in the text field.
  - **Clicked Item**: to use an attribute from the clicked row or chart part.  
    You can select any column from the view, even if it is not displayed in the widget.  
    Note that this choice is not available for **gauge** links, as there is no clickable sub-item.
  - **Dashboard parameter**: to use a global parameter from the dashboard as a contextual value for the target dashboard parameter.
    Select the dashboard parameter from the list.
  - **User**: to use an attribute related to the currently connected user. Select the attribute from the list.

![Another Dashboard link](./images/link_dashboard_cfg.png "Another Dashboard link")

#### Search Detail Dialog Box

New in **Curie R1**

This link is only available for **gauge** widgets and only when the value aggregation function is set to `Count`. It will display the list of items involved in the search count as a table in a dialog box.
You can further specify the columns to be displayed in the table, depending on the counted entity type (account, etc.)

![Search detail dialog box config](./images/link_dialog_cfg.png "Search detail dialog box config")

When clicking the gauge figure, a dialog box appears that lists the items in the count as a table with the selected columns. In this simple case, a list of 328 orphan accounts.

![Search detail dialog box](./images/link_dialog.png "Search detail dialog box")

From this dialog box, you can:

- Export this list as a CSV or Excel file
- Open the search page for the counted entity (_e.g._ accounts) with the search criteria already set to match the gauge search, so that it will display the same list and give access to all the search analytics. In the case above, the search page is opened on orphan accounts, and displays the same 328 items.

![Open in search page](./images/link_dialog_search.png "Open in search page")

#### External Link

- Type in **Link URL** a valid URL for the link. The URL must start with http:// or https://. It can contain dynamic placeholders.
- Select from the **Insert variable** menu a variable in the URL text. All valid variables are listed, and grouped in two categories: **Clicked Item attributes** and **Mashup parameters**. The actual variable values will be url-encoded in the URL text (_e.g._ space => %20)

![External link](./images/link_url_cfg.png "External link")

Example of an actual URL:

![Actual url](./images/link_url_test.png "Actual url")

## Configure Dashboard Resources

Dashboard **resources** are **layouts**, **chart styles** and **indicator styles**.
Dashboard resources are defined in `.dashres` files that are stored in the project in the `webportal/resources` directory.

Two dashboard resources are provided by default:

- **default**: which defines the default layout templates, chart styles and indicator styles
- **flatcolors**: which defines additional "flat colors" indicator styles and chart styles.

Additional dashboard resources can be defined and stored in other `.dashres` files.

This is the XML structure of the `.dashres` file:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE DashboardResourceBundle PUBLIC "-//Brainwave//DTD DashboardResourceBundle 1.0//EN" "urn:brainwave:igrc:dtds/dashres.dtd">
<DashboardResourceBundle name="default" displayname="Dashboards default resources">
    <layoutTemplates defaultId="1-1">
        <LayoutTemplate id="1-1" label="1 cols 100%" nbGridCols="1" nbCols="1" spans="1" />
        <LayoutTemplate id="2-1" label="2 cols 50/50%" nbGridCols="2" nbCols="2" spans="1 1" />
    </layoutTemplates>
    <indicatorStyles defaultId="default">
        <IndicatorStyle id="default" label="Default blue" valueColor="#1F76C7" warningColor="#ED9748" criticalColor="#FF6C6C" roundedTip="true" thickness="3" />
        <IndicatorStyle id="blueGrad" label="Blue gradient" valueColor="#008EA8 #006577" warningColor="#FFC268" criticalColor="#FF4C50" roundedTip="false" thickness="0" />
    </indicatorStyles>
    <chartStyles defaultId="default">
        <ChartStyle id="default" label="Blue" serieColors=" #1F76C7" />
        <ChartStyle id="blueScale" label="Blue scale" serieColors="#175A97 #1F76C7 #2182DA #61A7E7 #AAD0F2" />
    </chartStyles>
</DashboardResourceBundle>
```

### Dashboard Layout Templates

Dashboard layout is based on a grid system, with a fixed number of columns of equal width and a variable number of rows, depending on the content.
Each widget in the dashboard can span a given number of columns and rows.

A dashboard layout template defines the following characteristics of the layout:

| attribute      | meaning                                                                                                                                                                                                                      |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **id**         | unique identifier for the layout                                                                                                                                                                                             |
| **label**      | the label to be displayed in the configuration wizard                                                                                                                                                                        |
| **nbGridCols** | the number of columns in the underlying grid                                                                                                                                                                                 |
| **nbCols**     | the number of visible columns. This is an information-only field; you can use the same value as nbGridCols                                                                                                                   |
| **spans**      | the default layout sequence of successive widgets, separated by white spaces. Width and height are indicated as WxH, and 1 is a shortcut for 1x1. Additional widgets will get the layout of the last item in the sequence |

For example, suppose you would like to define a layout template with 3 widgets on the top and a large widget at the bottom, as shown below.

![Layout1](./images/res_layout1.png "Layout1")

In this case, you need a 3-column grid template, with three widgets of 1x1 width and height on the first row, plus one large widget that spans 3 grid columns x 2 grid rows on the next row.
This sequence will be encoded as: `1 1 1 3x2`

The final template definition will be:

```xml
<LayoutTemplate id="3-2" label="3 col + full" nbGridCols="3" nbCols="3" spans="1 1 1 3x2 " />
```

Note that additional widgets will inherit the layout of the last item in the template, in this case 3x2.
You can define as many templates as you like in different files.

### Indicator Styles

An indicator style allows you to define the look and feel of dashboard indicator widgets, mainly the color of the indicator ring.

The attributes of an indicator style are the following:

| attribute         | meaning                                                                                                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **id**            | unique identifier for the style                                                                                                                                            |
| **label**         | the label to be displayed in the configuration wizard                                                                                                                      |
| **valueColor**    | the color of the indicator ring. This color can be either a plain color in `#RRGGBB` notation, or a gradient color with two colors separated by a space. `#RRGGBB` `#RRGGBB` |
| **warningColor**  | not yet supported.                                                                                                                                                         |
| **criticalColor** | not yet supported                                                                                                                                                          |
| **roundedTip**    | not yet supported                                                                                                                                                          |
| **thickness**     | not yet supported                                                                                                                                                          |

The following example defines a style with a green to blue gradient ring color, from #00AA4D to #6A00F3.

```xml
 <IndicatorStyle id="greenBlue" label="Green blue gradient" valueColor="#00AA4D #6A00F3"  />
```

The indicator could look as follows:

![Indicator Style](./images/res_indstyle2.png "Indicator style")

> **Note:** It is recommended not to use warm colors (red and orange) for the ring color, so as not to be confused with the warning and critical threshold colors.

### Chart Styles

A chart style allows you to define the look and feel of dashboard charts, mainly series line and bar colors.

The attributes of a chart style are the following:

| attribute       | meaning                                                                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **id**          | unique identifier for the style                                                                                                             |
| **label**       | the label to be displayed in the configuration wizard                                                                                       |
| **serieColors** | space-separated list of series colors in `#RRGGBB` format. If there are more series than colors, the chart will cycle through the defined colors |

The following example defines a chart style with 5 series colors:

```xml
 <ChartStyle id="vivid" label="Vivid" serieColors="#01A9CD #FFB713 #F71B27 #6A6A6C #212B2C" />
```

The resulting configuration preview and chart will look as follows:

![Chart Style](./images/res_chartstyle_preview.png "Chart style")

![Chart Style](./images/res_chartstyle.png "Chart style")

## Project-based Dashboards

There are two kinds of dashboards for different usages: **user dashboards** and **project-based dashboards**.

**User dashboards** are the regular dashboards created, modified and managed by **dashboard designers** through the web interface.
These dashboards can then be shared for use with target groups of end-users.

User dashboards are intended to be specific to a given customer and modified by customers to fit their specific needs.
For this reason, they are stored in the database on each customer's environment.
User dashboards must be manually transferred between the different customer environments (_e.g._ dev, test, production) or backed up using the export/import feature.

**Project dashboards**, on the other hand, are not specific to a customer; they address a common need and are delivered inside a facet or an app, just like pages, views and rules.
Project dashboards are stored as `.dashboard` files inside the project in the specific `webportal/dashboards/` folder or a facet sub-folder.

Contrary to user dashboards, project dashboards will be shared between customers that use the same app or facet and across different environments - dev, test, production - of the same customer, as long as the web portal references the same project.
Project dashboards included in the project will automatically appear in the web portal to authorized users, without having to be manually imported.

### Create a Project Dashboard

Project dashboards are created and edited by dashboard **developers**, _i.e._ users with the **developer** role.

To create a project dashboard, a developer must do the following:

- Click on the **Create** button in the Dashboard manager. The dashboard creation wizard is displayed.
- First give a title to the dashboard in the **Title** box
- Check the **Save to project** option (1). This option is only available if you have the new "developer" role (see below).
- A default file name is proposed, based on the dashboard title. You can change the file name (2).
- The file is stored by default in the `webportal/dashboards` folder.
  You can specify a different subfolder, for example to include the dashboard in a specific facet, by clicking the folder icon (3).
- View & edit the dashboard in the portal as for user dashboards.
- Don't share the dashboard yet if it's intended to be distributed to different customers, through a facet or an app, because the target audience is not known at development time.

Any dashboard developer can edit any project dashboard, even without being its author.

![Dashboard Project](./images/dash_proj.png "Creating project dashboard")

**Note:** The **designer** and **developer** roles, dedicated to dashboards, are provided in the `bw_dashboards` facet, which is included in the **Identity Analytics version 1.4** add-on.

The Identity Analytics add-on is available on the [Radiant Logic file sharing site](../../downloads/01-downloads#add-ons-facets) (Customer Downloads > Facets > IAP).

### Working with Project Dashboards

All project dashboards in the current project are visible to developers and technical administrators in the dashboard manager, with a special "Project" type.

![Dashboard Project List](./images/dash_proj_list.png "Dashboard list")

Once the project is deployed to the web portal, a technical administrator - or a developer if he has access to the target platform - must share project dashboards with the target audience.
Each time the project dashboard is modified, it will be automatically updated on the web portal and the sharing configuration will be preserved.

Dashboard designers that don't also have the developer role cannot modify project dashboards.
This is because project dashboards can be modified in the project and will be automatically updated when the project is redeployed.
This would cause conflicts if the project dashboards were otherwise modified by designers on the target platform.
If a designer would like to modify a project dashboard, he can duplicate the dashboard as a user dashboard, by un-checking the **Save to project** option in the duplicate wizard.

Similarly, a developer can transform an existing user dashboard into a project dashboard, by duplicating the user dashboard and checking the **Save to project** option in the duplicate wizard.

Note that project dashboards are excluded from the export feature.

#### Project Dashboard Lifecycle

To sum up, the typical lifecycle of a project dashboard would be the following:

First time:

- a developer creates the project dashboard on the dev environment web portal
- the project is deployed to a test environment web portal
- a technical administrator shares the project dashboard with a test audience
- the project dashboard is tested
- the project is deployed to a production environment web portal
- a technical administrator shares the project dashboard with the actual target audience
- end-users can access the project dashboard

Each upgrade:

- a developer modifies the project dashboard on the dev environment web portal
- the project is deployed to a test environment web portal
- the project dashboard is tested
- the project is deployed to a production environment web portal
- end-users can access the modified project dashboard

#### Detailed Personae and Permissions

The table below lists the detailed authorizations for each persona:

| Persona                     | Permissions                                                                                                                                                                                                                                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Designer**                | Can create (and own), modify and delete user dashboards<br><br>Can duplicate a user dashboard as another user dashboard<br><br>Can share user dashboards<br><br>Cannot create a new project dashboard nor modify an existing one                                                                              |
| **Developer**               | All designer permissions plus...<br><br>Can view, create, modify and delete all project dashboards (no ownership)<br><br>Can duplicate user or project dashboards to either user or project dashboards<br><br>Can optionally set default sharing parameters for any project dashboard (which can be overridden) |
| **Technical administrator** | Can list and view all project and user dashboards<br><br>Can import / export all user dashboards<br><br>Can modify sharing parameters for user and project dashboards<br><br>Can delete any dashboard<br><br>Cannot duplicate dashboards                                                                     |
| **End user**                | Can view all project and standard dashboards shared with him<br><br>Cannot create, modify or delete dashboards                                                                                                                                                                                               |

## Using Pre-Computed KPIs

Most KPIs in the Dashboards use rules to compute the number to display as the KPI value.
It provides an easy way to build KPIs, but it can result in slow dashboards if they contain a lot of KPIs to compute.

To improve performance, it's possible to pre-compute the KPIs during the execution plan, and store the results in a metadata.
In the dashboards, you can switch to the advanced mode to call a view instead of a rule.
This allows you to directly retrieve the number to display instead of computing it, and should significantly improve performance.

### Example

Below is an **example** of what has been done for the `Active Directory Controls` dashboard, using this method (update available with `IDA 3.7` or later releases).

#### Metadata definition

The goal is to have a metadata linked to the `Repository` concept that will store the number of defects for each control.
There is one for the number of account defects, and one for the number of control defects.
They are very similar; let's use the one on the accounts as an example.

![Metadata Definition](images/bwa_repo_accounts_kpis_md_definition.jpg)

The `bwa_repo_accounts_kpis` metadata definition is:

- `Display metadata attributes in the following concept in the view editor` option checked
- Single concept set to `Repository`
- `Display also in smartsearch interface, in mashup chart configuration, and in the rule editor` option checked
- It uses three fields:
  - `controlCode`: the code of the associated control
  - `controlName`: the name of the associated control
  - `nb`: the number of account defects for the current control & repository

#### Metadata computation

The metadata is computed during the execution plan, after the controls (because we want to store the number of defects):

![Metadata computation](images/bwa_repo_accounts_kpis_md_computation.jpg)

We need two other values in addition to the control & number of defects:

- The `Repository attribute (uid)`, to link the metadata value to each repo
- The `Sub key attribute`, to allow the metadata to have more than one value per repo: one for each control

> Note that using a metadata computed during the execution plan means that:  
> The KPIs won't be refreshed if the data changes on an existing timeslot (for example if an account is manually reconciled).  
> The KPIs are only updated when creating a new timeslot.

The view used to compute the metadata looks like this (`bwa_repo_accounts_kpis` if you want to check it):

![Metadata computation view](images/bwa_repo_accounts_kpis_computation_view.jpg)

#### View used by the Dashboard

Then we need a view that will be used by the Dashboard, for example for the accounts KPIs (`bwa_repo_accounts_kpis_results`):

![Results view for the Dashboard](images/bwa_repo_accounts_kpis_results_view.jpg)

A few things of note:

- View parameters
  - The only required parameter is the `controlCode`
  - The dashboard will automatically filter on the repo, but it's good to have it for testing
- You **must** have an `inner join` in the view even if it returns no attributes
  - This is because of a limitation in the engine
- You **must** check the `Allow the selection of the view in the portal` option in the `Execution` tab
  - ![View publication in the portal](images/bwa_repo_accounts_kpis_view_publication.jpg)
- Enable both of those options:
  - `Toggle repository perimeter visibility`
  - `Toggle web portal search anchor`
  - ![View toggles](images/bwa_repo_accounts_kpis_view_toggles.jpg)

#### Dashboard update

Update an existing KPI to use the results view defined earlier.
Go to the `Value` tab of the KPI, select `Sum` as the aggregation function and click on `Advanced`:

![Sum aggregation & Advanced mode](images/bwa_repo_accounts_kpis_sum.jpg)

This will allow you to pick the view with the saved KPIs metadata and to pass the correct parameter values:

![alt text](images/bwa_repo_accounts_kpis_view.jpg)

Save the dashboard and remember to launch a new execution plan.
