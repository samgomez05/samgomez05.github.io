---
layout: single
title:  "Enhancement Two: Algorithms"
date:   2025-03-24
categories: capstone
permalink: /Enhancement\ Two/
author_profile: false
classes: wide
---

## Briefly describe the artifact. What is it? When was it created?

For the second artifact, I elected to keep working on the Inventory Management application that I was using for the first category in this class.


## Justify the inclusion of the artifact in your ePortfolio. Why did you select this item? What specific components of the artifact showcase your skills and abilities in software development? How was the artifact improved?

I chose to use the same artifact I was using in the previous module as I’m hoping to get a fully polished application that allows me to demonstrate all the competencies of a good Software Engineer. I want my application to be as polished as possible so that my current and future employers see these competencies as well as my dedication to the application and the evolution of it over time. 

For this module, I wanted to focus on having a filter and search algorithm to allow the end-user to administer an item in a simpler way. Before implementing the changes I had to go through this week, described in the following question, an application user would have to go through a list of items, scrolling through all, in order to modify the quantities.

This week I met all of the course outcomes that I wanted to implement on my application. I felt more attuned with it and felt it become easier to implement these changes and look forward to improving upon it as best I can to showcase my engineering skills as a Computer Scientist.


## Reflect on the process of enhancing and modifying the artifact. What did you learn as you were creating it and improving it? What challenges did you face?

Through the past two weeks, I focused on getting more knowledge on how to best implement the course outcomes for this week. I came to realize that the way that my Views were implemented in the application were not optimized for the implementation of the changes that I wanted to do and therefore I had to look back at how best to change the application to do so. 

I ended up doing some major refactoring work to implement better classes that were suited for the task, rather than using the ones that I implemented while I was in CS-360. The first change was to implement an extension of the ‘SimpleCursorAdapter’ class available to allow the simplification of how each of the items in the views were displayed. This required a lot of research and videos that impacted how much I was able to accomplish last week but ended up working out incredibly well for everything else I had to do this week. 

```java
    public class InventoryAdapter extends SimpleCursorAdapter {

        private final InventoryDatabaseHelper dbHelper;
        private final LayoutInflater inflater;

        public InventoryAdapter(Context context, int layout, Cursor c, String[] from, int[] to, int flags, InventoryDatabaseHelper dbHelper) {
            super(context, layout, c, from, to, flags);
            this.dbHelper = dbHelper;
            this.inflater = LayoutInflater.from(context);
        }
        ...
    }
```

Per-item binding in InventoryAdapter:
```java
    // InventoryAdapter.java
    @Override
    public void bindView(android.view.View view, Context context, Cursor cursor) {
        TextView itemNameTextView = view.findViewById(R.id.itemNameTextView);

        TextView itemDescriptionTextView = view.findViewById(R.id.itemDescriptionTextView);

        TextView itemQuantityTextView = view.findViewById(R.id.itemQuantityTextView);

        Button itemAddButton = view.findViewById(R.id.itemAddButton);

        Button itemSubtractButton = view.findViewById(R.id.itemSubtractButton);
    }
```


I extracted most of the per-item functionality from the ‘MainActivity’ class into this new ‘InventoryAdapter’ class and was able to do so so that this would iterate over a provided cursor for each of the items. Two major improvements resulted from this. First, the performance of the application would theoretically be improved as an adapter would only keep in memory those objects that are displayed, and not the entire list. Additionally, the ‘ListView’ and ‘GridView’ ‘ViewGroups’ were a lot easier and a lot less code to implement as they are built with the adapter in mind, simplifying the way these are displayed. 

```java
    // MainActivity.java
    private void displayDatabaseItems() {
        // Get all inventory items
        Cursor cursor = inventoryDbHelper.getAllInventoryItems();

        // Map db columns to view
        String[] fromColumns = {"item_name", "item_description", "item_quantity"};
        int[] toViews = {R.id.itemNameTextView, R.id.itemDescriptionTextView, R.id.itemQuantityTextView};

        // Check what layout is visible - list or grid
        int layoutId = R.layout.item_view_list;
        if (listView.getVisibility() == View.GONE) {
            layoutId = R.layout.item_view_grid;
        }

        // Create adapter to display each item received by the cursor
        inventoryAdapter = new InventoryAdapter(this, layoutId, cursor, fromColumns,
                toViews, 0, inventoryDbHelper);

        inventoryAdapter.sortByName(sortAscending);

        // Set data to listView, everything in adapter
        if (listView.getVisibility() == View.VISIBLE) {
            listView.setAdapter(inventoryAdapter);
        } else {
            gridView.setAdapter(inventoryAdapter);
        }

    }
```

Once I got to this point, the search, sort, and filter functionality was significantly easier to implement and display. I adapted the InventoryAdapter class to have methods for each of these so that it was a matter of implementing a little more code to the main class and views to get this to work. In this snippet of code, the applica


```java
    // MainActivity.java
    private void initSearchView() {
        // Finds the UI element in the display and `binds` to it.
        SearchView searchView = (SearchView) findViewById(R.id.searchView);
        
        // Instantiates a query listener to take action based on changes to the UI element.
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            
            // "Necessary" override that has to be instantiated as part of 
            // the OnQueryTextListener - no function.
            @Override
            public boolean onQueryTextSubmit(String search) {
                return false;
            }

            // Search on key strokes. Works on both List and Grid view, depending 
            // on which is active.
            @Override
            public boolean onQueryTextChange(String search) {
                Cursor cursor = inventoryDbHelper.searchInventoryItems(search);
                InventoryAdapter adapter = (InventoryAdapter) listView.getAdapter();
                if (listView.getVisibility() != View.VISIBLE) {
                    adapter = (InventoryAdapter) gridView.getAdapter();
                }
                adapter.swapCursor(cursor);
                return false;
            }
        });
    }
```

Needed to add this functionality in the XML layout view for MainActivity:

```xml
    <!-- activity_main.xml -->
    <androidx.appcompat.widget.SearchView
        android:id="@+id/searchView"
        ...>

    </androidx.appcompat.widget.SearchView>

    <LinearLayout
        android:id="@+id/filterView"
        ...>


        <Button
            android:id="@+id/filterByElectronics"
            .../>

        <Button
            android:id="@+id/filterByOffice"
            .../>

        <Button
            android:id="@+id/resetFilter"
            .../>

    </LinearLayout>
```

The application now has a pop-up menu, implemented last week, with the ability to display a search and filter view, and sorting, implemented this week, where the search bar allows the user to search for and, in real-time, display the items they need to modify and do so. This same menu option drops in a few buttons to allow the user to filter for a few items based on a tag that is provided to them and automatically just filter out those that don’t match that tag. 

```java
    // InventoryAdapter.java
    public void filter(String category) {
        // Retrieve the filtered cursor from the database
        Cursor filteredCursor = dbHelper.getFilteredInventoryItems(category);
        changeCursor(filteredCursor);
    }
```

For now, only two are provided, ‘electronics’ and ‘office supplies’, though I plan on not having these fixed but only for demonstration purposes this week. I also adapted the inventory database to house a per-item description to allow the user to understand what the item is.

<p style="text-align: center"><img src="/assets/images/enhancementTwo_search_and_filters.png"></p>


Here are some of the resources I used to educate myself through the development of this enhancement:
- [SimpleCursorAdapter](https://developer.android.com/reference/android/widget/SimpleCursorAdapter)
- [SearchView](https://developer.android.com/reference/android/widget/SearchView)
- [Filter & Search List View Android Studio Tutorial (Video)](https://youtu.be/M73Vec1oieM?si=3KwC-6GoL-weI3Nc)

Project with changes linked below:
- [Enhancement Two](https://1drv.ms/u/s!AkQ5hcKSstgUmlps3IJAKPi2cF0P?e=hECZE6)