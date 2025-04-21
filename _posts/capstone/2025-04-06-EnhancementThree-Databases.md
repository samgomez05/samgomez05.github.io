---
layout: single
title:  "Enhancement Three: Databases"
date:   2025-04-06
categories: capstone
permalink: /Enhancement\ Three/
author_profile: false
classes: wide
---

## Briefly describe the artifact. What is it? When was it created?

For the third artifact, I elected to keep working on the Inventory Management application that I was using for the first category in this class. It is an inventory application that I created here at SNHU in the CS-360 class.

## Justify the inclusion of the artifact in your ePortfolio. Why did you select this item? What specific components of the artifact showcase your skills and abilities in software development? How was the artifact improved?

For this artifact, I realized and learned about RecyclerView and the benefits of implementing it, such as it being able to recycle the adapter positions for better memory management. Since it seems better suited for the task, I refactored the InventoryAdapter class to extend the RecylcerViewer functionality so that I would not need to use both the ListView and GridView implementations in ‘activity_main.xml’. 

```java
    public class InventoryAdapter extends RecyclerView.Adapter<InventoryAdapter.ViewHolder> {
        ...

        public InventoryAdapter(List<InventoryItem> inventoryList, boolean isListView, FirebaseDatabaseHelper firebaseDatabaseHelper) {
            mInventoryList = inventoryList;
            this.isListView = isListView;
            this.mFirebaseDatabaseHelper = firebaseDatabaseHelper;
        }

        public class ViewHolder extends RecyclerView.ViewHolder {
            ...

            public ViewHolder(@NonNull View itemView) {
                super(itemView);
                ...
            }
        }
    }
```

By using RecyclerView, I could then simplify how each of the items is "bound" and, similarly to the refactor done in Enhancement Two, this allows a per item binding. Unlike previous binding however, this simplifies the amount of code necessary to get a list and grid view working and further modularizes the application.

```java
    // InventoryAdapter.java
    @Override
    public void onBindViewHolder(@NonNull InventoryAdapter.ViewHolder holder, int position) {
        InventoryItem currentItem = mInventoryList.get(position);

        holder.itemNameTextView.setText(currentItem.getName());
        holder.itemDescriptionTextView.setText(currentItem.getDescription());
        holder.itemQuantityTextView.setText(String.valueOf(currentItem.getQuantity()));

        // Binds to the increment button (+) on the item view
        holder.itemAddButton.setOnClickListener(v -> {...});

        // Binds to the decrement button (-) on the item view
        holder.itemSubtractButton.setOnClickListener(v -> {...});

        // Creates a per-item listener that allows clickability to inflate a dialog to edit said item
        holder.itemView.setOnClickListener(v -> showEditItemDialog(currentItem, position, holder));

    }
```

I also found that by doing this, I was able to resolve a ‘shake’ or ‘jump’ that occurred while updating the item quantities. 

Finally, and most importantly, it removes the Cursor extension as I’m no longer using an on-device database. I elected to use the Firebase database option that is available to Android developers throught the Android Studio wizard. 

```java
    public class FirebaseDatabaseHelper {


        private final DatabaseReference databaseReference;

        public FirebaseDatabaseHelper(DatabaseReference reference) {
            this.databaseReference = reference;
        }
    }
```

This simplified a lot of the work that I needed to do to develop for another database as it is easier to implement. I added various entries into the database and changed all the methods and classes that required the on-device database implementation. Finally, to load the inventory I only needed to call all items from the database using the `loadInventory()` method.

```java
    private void loadInventory() {
        firebaseDbHelper.fetchItems(new ValueEventListener() {
            @SuppressLint("NotifyDataSetChanged")
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                inventoryItemList.clear();
                tagList.clear();
                for (DataSnapshot itemSnapshot : snapshot.getChildren()) {
                    InventoryItem item = itemSnapshot.getValue(InventoryItem.class);
                    if (item != null) {
                        // Set item id from key to populate missing id data
                        item.setId(itemSnapshot.getKey());
                        if (!tagList.contains(item.getTag())) {
                            tagList.add(item.getTag());
                        }
                        inventoryItemList.add(item);
                    }
                }

                // Applies array created above to inventoryAdapter implementation to refresh
                inventoryAdapter.setItems(new ArrayList<>(inventoryItemList));
                initSpinner();
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {
                Toast.makeText(MainActivity.this, "Failed to load data", Toast.LENGTH_SHORT).show();
            }
        });
    }
```

I also added the remove functionality to ensure that the end-user was able to remove items that were unnecessary. This was done with an `ItemTouchHelper`. I elected to make this the method to use as it is intuitive and somewhat popular in plenty of other applications (think of how you delete a message thread or an email in their respective applications).

```java
    // MainActivity.java - setUpItemTouchHelper()
    @Override
    public void onSwiped(@NonNull RecyclerView.ViewHolder viewHolder, int direction) {
        // Get the position of the item
        int position = viewHolder.getAdapterPosition();
        InventoryItem itemToDelete = inventoryAdapter.getItemAt(position);

        // Create a dialog to confirm deletion to prevent deleting items accidentally
        AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
        builder.setTitle("Delete Item");
        builder.setMessage("Are you sure you want to delete \"" + itemToDelete.getName() + "\"?");

        // UI element to confirm removal of item. Confirming pushes updates to Firebase DB and inventoryAdapter
        builder.setPositiveButton("Yes", (dialog, which) -> {
            firebaseDbHelper.deleteItem(itemToDelete.getId());
            inventoryItemList.removeIf( v -> v.getId().equals(itemToDelete.getId()));
            inventoryAdapter.removeItem(itemToDelete);
        });

        // Cancel
        builder.setNegativeButton("No", (dialog, which) -> {
            dialog.dismiss();
            recyclerView.post(() -> {
                // Reverse animation added for visual appeal
                RecyclerView.ViewHolder holder = recyclerView.findViewHolderForAdapterPosition(position);
                if (holder != null) {
                    View itemView = holder.itemView;
                    ObjectAnimator animator = ObjectAnimator.ofFloat(itemView, "translationX", itemView.getTranslationX(), 0);
                    animator.setDuration(250);
                    animator.start();
                }

                inventoryAdapter.notifyItemChanged(position);
                itemTouchHelper.attachToRecyclerView(null);
                itemTouchHelper.attachToRecyclerView(recyclerView);
            });
        });

        builder.setCancelable(false);
        builder.show();
    }
```

<p style="text-align: center"><img src="/assets/images/enhancment_three_database_swipe_to_delete.png"></p>


After implementation, I found that the way my current implementation of this was done caused some mishaps when deleting an item after a filter or search was done. I found that this was the case as the item being deleted was not really the item in the position on the adapter but the position in the list. To fix this, I moved some of the logic in `MainActivity` to `InventoryAdapter`, so that the adapter could determine which of the items was the correct one to delete.

```java
    // InventoryAdapter.java constructor
    // Added inventoryList list as a parameter to ingest the list, filtered, searched, or default.
    public InventoryAdapter(List<InventoryItem> inventoryList, boolean isListView, FirebaseDatabaseHelper firebaseDatabaseHelper) {
            mInventoryList = inventoryList;
            this.isListView = isListView;
            this.mFirebaseDatabaseHelper = firebaseDatabaseHelper;
        }

```

The following methods allowed the fix of the aforementioned issue and can be seen in the `loadInventory()` method above.

```java
    // InventoryAdapter.java
    public InventoryItem getItemAt(int position) {
        return mInventoryList.get(position);
    }

    public void removeItem(InventoryItem item) {
        int index = mInventoryList.indexOf(item);
        if (index >= 0) {
            mInventoryList.remove(index);
            notifyItemRemoved(index);
            notifyItemRangeChanged(index, mInventoryList.size());
        }
    }
```


## Did you meet the course outcomes you planned to meet with this enhancement in Module One? Do you have any updates to your outcome-coverage plans?

I did end up migrating the database implementation for the inventory items database and did refactor as much as necessary to adapt to it. However, I still need to implement the ability to authenticate with Firebase Auth to grant access to the Firebase database.

Firebase Auth allows a setup of rulesets that can be defined as the correct 

## Reflect on the process of enhancing and modifying the artifact. What did you learn as you were creating it and improving it? What challenges did you face?

For this category I learned how to implement the RecyclerView adapter and Layout managers in order to replace most of the overhead that I needed to have both list and grid views. I had to learn about the Firebase realtime database and how best to implement it. This was challenging as the on-device implementation was a lot easier. This was a challenge for sure but I think I’m a much better developer for it.


Here are some of the resources I used to educate myself through the development of this enhancement:
- [Get Started with Realtime Database](https://firebase.google.com/docs/database/flutter/start)
- [Get Started with Firebase Authentication on Android](https://firebase.google.com/docs/auth/android/start)
- [Create dynamic lists with RecyclerView](https://developer.android.com/develop/ui/views/layout/recyclerview)
- [ItemTouchHelper](https://developer.android.com/reference/androidx/recyclerview/widget/ItemTouchHelper)
- [Add spinners to your app](https://developer.android.com/develop/ui/views/components/spinner)

Project with changes linked on the sidebar.
