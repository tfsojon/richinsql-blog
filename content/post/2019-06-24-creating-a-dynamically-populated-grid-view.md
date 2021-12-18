---
title: Creating a dynamically populated Grid View
date: 2019-06-24T16:00:12+01:00
author: Rich
layout: post
permalink: /creating-a-dynamically-populated-grid-view
categories:
  - 'csharp'
tags:
  - ASP.net
  - GridView
  - VisualStudio
---
I haven't published a post here in what feels like an eternity, I came across a problem recently and wasn't sure how to go about writing the code, what I had was a form which consisted of a number of drop-down boxes and I needed to get the values of the drop-down boxes into a GridView, then loop that GridView to insert that data into a database.

The problem was I also needed to allow for a row to be removed from the Grid View if the user decided the data selected was no longer required.

### The Web Form

I will assume you know how to create a form in Visual Studio mine looked like something like this in HTML

```
&lt;div class="row" style="margin-top:10px"&gt; 
    &lt;div class="col-md-6"&gt;
        &lt;asp:DropDownList id="DDL_Products" runat="server"&gt;
            &lt;asp:ListItem Text="Product1" Value="1"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Product2" Value="2"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Product3" Value="3"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Product4" Value="4"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Product5" Value="5"&gt;&lt;/asp:ListItem&gt;
        &lt;/asp:DropDownList&gt; 
    &lt;/div&gt; 
    &lt;div class="col-md-6"&gt;
        &lt;asp:DropDownList id="DDL_Manufacturers" runat="server"&gt;
            &lt;asp:ListItem Text="Manufacturer1" Value="1"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Manufacturer2" Value="2"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Manufacturer3" Value="3"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Manufacturer4" Value="4"&gt;&lt;/asp:ListItem&gt;
            &lt;asp:ListItem Text="Manufacturer5" Value="5"&gt;&lt;/asp:ListItem&gt;
        &lt;/asp:DropDownList&gt; 
    &lt;/div&gt; 
&lt;/div&gt;
&lt;div class="row" style="margin-top:10px"&gt; 
    &lt;div class="col-md-12"&gt;
    &lt;asp:Button id="Btn_AddRow" runat="server" OnClick="Btn_AddRow_Click" Text="Submit" /&gt;
    &lt;/div&gt;
&lt;/div&gt;
```

Which should look something like this on the page

![](/img/DynamicGridView-1.png)

**_I am using Bootstrap for the framework_**

The form has a button on it called Btn\_AddRow which has an on click event of Btn\_AddRow_Click, the code for this method looks like this, we will get to that in a moment but essentially we are calling another method when the button is clicked.

```
        protected void Btn_AddRow_Click(object sender, EventArgs e)
        {
            AddNewRows();
        }
  ```

### The GridView

On the same page, I have an asp GridView control this is where the products that I select from the drop-down list will appear when the button is clicked, On initial page load the GridView will not show up on the page as the flag AutoGenerateColumns is set to false and there is no data bound to the GridView control at this point.

```
        &lt;div class="row" style="margin-top:10px"&gt;
            &lt;div class="col-md-12"&gt;
                &lt;asp:GridView ID="GV_OrderedProducts" AutoGenerateColumns="false" OnRowDeleting="GV_OrderedProducts_RowDeleting" CssClass="table table-bordered" runat="server"&gt;
                    &lt;Columns&gt;
                        &lt;asp:BoundField DataField="Manufacturer" HeaderText="Investigation Required" /&gt;
                        &lt;asp:BoundField DataField="ManufacturerID" Visible="false" /&gt;
                        &lt;asp:BoundField DataField="Product" HeaderText="Requesting Facility" /&gt;
                        &lt;asp:BoundField DataField="ProductID" Visible="false" /&gt;
                        &lt;asp:CommandField ShowDeleteButton="True" ControlStyle-CssClass="btn btn-sm btn-danger" DeleteText="Remove Row" /&gt;                     
                    &lt;/Columns&gt;
                &lt;/asp:GridView&gt;
            &lt;/div&gt;
        &lt;/div&gt;
  ```

### Initialize The Grid

When the page first loads, I need to Initialize the GridView to do this I am going to create a new ViewState called dtSource then create a DataTable which will contain the columns of my Gridview, populating the ViewState with the data table as the source and binding it to the GridView.

```
        private void InitializeGrid()
        {
            if (ViewState["dtSource"] == null)
            {
                DataTable dtSource = new DataTable();
                dtSource.Columns.Add("Manufacturer");
                dtSource.Columns.Add("ManufacturerID");
                dtSource.Columns.Add("Product");
                dtSource.Columns.Add("ProductID");      
                ViewState["dtSource"] = dtSource;

                GV_OrderedProducts.DataSource = dtSource;
                GV_OrderedProducts.DataBind();
            }
        }
  ```

### Add New Rows

I now need a method to handle the events that will take place when I click the button to add a new row to the GridView.

  * I have created a new DataTable using the ViewState.
  * I have then created a new DataRow and called the NewRow() method
  * Now I need to pass the data I would like to bind to the columns within that row, here I am passing the SelectedItem and SelectedValue from the DropdownList, the SelectedValue is hidden and will be used when inserting the data, the SelectedItem is to visually tell the user which Product & Manufacturer they have selected.
  * Finally, the GridView is rebound using the DataTable as the source.

```
    private void AddNewRows()
    {
       
        DataTable dtSource = ViewState["dtSource"] as DataTable;
        DataRow dr = dtSource.NewRow();
        dr["Manufacturer"] = DDL_Manufacturers.SelectedItem;
        dr["ManufacturerID"] = DDL_Manufacturers.SelectedValue;
        dr["Product"] = DDL_Products.SelectedItem;
        dr["ProductID"] = DDL_Products.SelectedValue;

        dtSource.Rows.Add(dr);
 
        GV_OrderedProducts.DataSource = dtSource;
        GV_OrderedProducts.DataBind();
    }
```

![](/img/GridViewNewRows.gif)

### On Row Deleting

This is the method that will handle the removal of the row from the GridView essentially what is happening here is we are creating a DataTable with the current data from the ViewState then we are getting the Row Index of the row that has been clicked then simply deleting that row and then rebinding the GridView using the DataTable and updating the ViewState.

```
    protected void GV_OrderedProducts_RowDeleting(object sender, GridViewDeleteEventArgs e)
    {
        int index = Convert.ToInt32(e.RowIndex);
        DataTable dt = ViewState["dtSource"] as DataTable;
        dt.Rows[index].Delete();
        ViewState["dtSource"] = dt;
        GV_OrderedProducts.DataSource = dt;
        GV_OrderedProducts.DataBind();
    }
```

![](/img/GridViewDeleteRows.gif)

### The Page Load

The Page Load method is where we will handle the Initialization of the GridView, remember that method InitializeGrid I created earlier, this is where that is called.

```
    protected void Page_Load(object sender, EventArgs e)
    {
        if (!IsPostBack)
        {
            InitializeGrid();           
        }
    }
```