---
layout: single
title:  "Enhancement One: Software Design and Engineering"
date:   2025-03-24
categories: capstone
permalink: /Enhancement\ One/
author_profile: false
classes: wide
---

## Briefly describe the artifact. What is it? When was it created?

This artifact I created for the CS-360 Mobile Development application class I took last year here at SNHU. It is an inventory management application written in Java. This application allows a user to manage items through a UI that allows adding new items and managing the quantities each item has.

## Justify the inclusion of the artifact in your ePortfolio. Why did you select this item? What specific components of the artifact showcase your skills and abilities in software development? How was the artifact improved?

I selected this artifact as I’m already in a Cloud Ops Engineering role where I do Java development. This artifact should showcase skills that I have learned in my time here at SNHU so that I’m able to present to my current and future employers. 

The artifact has multiple Java classes that, altogether, create an inventory application. Each has it’s own methods and implementations. Each of these was improved upon in my first week working on it. 

I refactored one of the functions that used to do the bulk of the work into two reusable functions that now generate two views, a list and a grid view. I implemented a dropdown menu that has a toggle option. It also houses the original logout option. 

```java
       private void populateListViewWithInventoryItems(ViewGroup view) {
        ...

        int layoutId = R.layout.item_view_list; // Defaults to `List`
        if (view.getTag().equals("item_view_grid")) {
            layoutId = R.layout.item_view_grid;
        }
        ...
    }
```

Both the list view and grid view now each has a per-item iterative generative function and each has it’s own view that is called upon to generate the items. I added an image (thought the logic here will need to be created to fill).

```java
    // Creates a per-item view and inflates them in the RelativeLayout
    private View createInventoryItemView(Cursor cursor, int layoutId) {
        ...

        TextView itemNameTextView = itemView.findViewById(R.id.itemNameTextView);
        TextView itemQuantityTextView = itemView.findViewById(R.id.itemQuantityTextView);
        ...
        // Add button and logic to increment item quantity
        Button addQtyButton = itemView.findViewById(R.id.itemAddButton);
        ...

        // Subtract button and logic to decrease item quantity
        Button subQtyButton = itemView.findViewById(R.id.itemSubtractButton);
        ...
    }

```

Using the following XML file:
```xml
        <ScrollView
        ...>
            <RelativeLayout
            ...>
                <LinearLayout
                .../>
                <GridLayout
                .../>
            </RelativeLayout>
        </ScrollView>
```

## Did you meet the course outcomes you planned to meet with this enhancement in Module One? Do you have any updates to your outcome-coverage plans?

I have met all of the course outcomes that I wanted to. I created javadocs for each of the methods in all classes, as well as added adequate comments where needed. I also developed the grid view that I wanted as well as the toggle for the list view. I set myself up for adding more dropdown menu items for filter and search in the next module.

I've also been able to develop test methods in the application so that functionality is tested througout furture development. I've done this by implementing both unit tests as well as Android instrumented tests. 

```sh
app/src/androidTest/java/com/snhu/cs360/inventoryapp/auth
└── LoginActivityTest.java
app/src/androidTest/java/com/snhu/cs360/inventoryapp/inventory
└── InventoryAdapterTest.java
app/src/androidTest/java/com/snhu/cs360/inventoryapp/MainActivityTest.java

app/src/test/java/com/snhu/cs360/inventoryapp/firebase
└── FirebaseDatabaseHelperTest.java
app/src/test/java/com/snhu/cs360/inventoryapp/inventory
└── InventoryItemTest.java
```

To aid with this, I've use the Junit, Mockito, and Espresso frameworks.

```gradle
androidTestImplementation libs.mockito.android
androidTestImplementation libs.rules
androidTestImplementation libs.runner
androidTestImplementation libs.ext.junit
androidTestImplementation libs.espresso.core
androidTestImplementation libs.espresso.contrib
```

```toml
junitJupiter = "5.11.4"
junitVersion = "1.2.1"
espressoCore = "3.6.1"
mockitoCore = "5.17.0"
#noinspection GradleDependency
runner = "1.6.1"
firebaseAuth = "23.2.0"
espressoContrib = "3.6.1"
rules = "1.6.1"
```


## Reflect on the process of enhancing and modifying the artifact. What did you learn as you were creating it and improving it? What challenges did you face?

Everything that I did to change the code was a challenge, but I learned a lot. From refactoring the function to be reusable, to understanding how to best implement some of the functionality that is required using Java and not Kotlin. 

Android documentation and most forums did end up serving up suggestions in Kotlin and translating that to Java was a challenge of it’s own. 

I went through multiple iterations over the course of the entire week, trying to accomplish everything that I had and learned that I may have underestimated how much time each feature takes to develop; something I will definitely bring forward into my career.


Here are some of the resources I used to educate myself through the development of this enhancement:
- [Build local unit tests](https://developer.android.com/training/testing/local-tests)
- [Build instrumented tests](https://developer.android.com/training/testing/instrumented-tests)
- [Espresso](https://developer.android.com/training/testing/espresso)
- [AndroidJUnitRunner](https://developer.android.com/training/testing/instrumented-tests/androidx-test-libraries/runner)
- [Mockito](https://site.mockito.org/#how)
- [How to write good tests](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)

The project with most of the changes: 
- [Enhancement One (Minus Tests)](https://1drv.ms/u/s!AkQ5hcKSstgUmllNa4wVEWhWoZYJ?e=AvEaCf)
