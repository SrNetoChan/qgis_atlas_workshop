# coruna workshop instruction

# Requirements

- QGIS 3.4 or later installed
- Data folder

# Preparation

1. Copy the file in the USB stick to your computer
2. Create a working copy of it elsewhere, so that you can change it without losing the original data

# Exercises

## Exercise 1 -Atlas usage with a grid

In this exercise, we will use the atlas functionality to cover all the Alaska at a 1:250.000 scale in a A3 layout pages.

3. Open the Alaska - Atlas Usage.qgs project.
4. Show how the map contains data from Alaska. Breafly explains the the layers it contains
5. Open the Alaska map layout
6. Show how all features are too small for a A3 page, making the map unreadable at a very small scale (show scale)
7. Go back to the project and show the grid layer. Enable it.
8. ?Talk about how to make a grid like that?
9. Go back to the layout, create a duplicate of the layout and call it "Alaska Atlas"
10. Go to the Atlas separator, and enable atlas generation. Select the number field as page name.
11. Turn the Atlas preview on. move a few atlas pages and notice that the map layout does not change.
12. Select the map item, enable the controlled by atlas option, and try again.
13. Change the margin to 0%
14. Turn of the grid layer on the map canvas (you could use the **Hide the coverage layer** option in atlas separator, but we will need it later.
15. Fix the scale bar. Set the the units to km, with 3 segments of 75 units (km). Center the legend if necessary.
16. Add a label with the page number to the Alaska label, something like (use the expression builder to do it):

```
    Alaska Atlas
    (Page [% @atlas_pagename %])
```

17. Click next in the preview toolbar and confirm that the label changes.
18. Go to the atlas tab and set the output name containing the page name "Alaska_atlas_pagenumber"

### Exporting the atlas pages


19. Export as png to an output folder
20. Show the folder populated with PNGs one for each map using the
21. Export as PDF, selecting the option to keep the files together if possible
22. Show that only one PDF was created with several pages.

END

### Create an overview map? (optional)

1. Use the same layout.
2. Add another map item to the canvas, a small one.
3. Show that, changing the map canvas in the project, it will affect the map you have already done.
4. Introduce to Presets and multiple style.
5. Create a preset to save the "fixed" style of the main map "Atlas - main map".
6. Disable all layers keep only the alaska area layer and the grid. Show that at this point the saved preset is no longer active.
7. Create a new style ("overview style") for the alaska layer, and set it to gray.
8. Show that you can swap between the default style and the new one.
7. Create another preset for the overview map
8. Show that you can quickly swap between the two presets
9. Go to the layouts and assign the respective presets to each map canvas.

## Exercise 2 - Follow features with atlas

We are going to
1. Close the previous layout.
2. Disable the grid layer
3. Open the Regions map. Show that it's a simple map of one of the regions, and that there are 25 more region's maps to do. Instead of creating it by hand, we will use atlas to do it for us.
5. Duplicate the layout and name it "Regions map"
6. In the Atlas tab, enable it, use the Regions layer as coverage layer. Use the name of the region as page name.
7. Select the map item, and make it controlled by the atlas.
8. Change the labels to automatically get the attributes in the coverage layer, namelly the region name and type, and the area.

   [% "NAME_2"%]'s [%"TYPE_2"%]

   Region's area is: [%to_int($area / 1000000)%] km2

9. Turn preview mode on, and show how regions get represented.
10. Highlight the fact that the legend gets huge in some cases, fix it by using a min and max width.

### Filter feature by atlas

1. Keep the atlas preview working.
Create a new style for the regions layer
2. Create a rule-based layer that uses the id to separate the current atlas feature from the rest with different symbols
4. Filter the region label, as it already shows in the map layout.
5. Create a new style for the airports. Notice that the  layer has a regions_fk id, use it to filter the airports in each region using again a rule-base symbol
3. Create a new style for settlements use a spatial operator to hide the one outside the atlas features (because there is no attribute relation that one can use)
5. Save a preset for this map. Call it "Regions-map"
6. Go back to the layout, set the new preset and show a preview of some region maps.


## Exercise 4 - Make the page orientation adapt to the orientation of the region

1. From the previous exercise, notice that the wide page format does not work well with tall regions, for example page number 9 haines.
2. Make the page orientation change depending on the region width and height.
3. set the following expression on the page layout.

   ```java
   CASE WHEN bounds_height( @atlas_geometry ) > bounds_width(@atlas_geometry) THEN
   'portrait'
   ELSE
   'landscape'
   END
   ```

4. We need to fix the map and other items to fit the page. We could set the same type of conditional behaviour for when the map is in portrait or landscape, but we can do even better, we can make the items size and position depend of their distance to the page itself, it the page changes, they will follow.

   Note that the page changes around the top-left corner.

5. Lets start with the map item. The position of the top-left corner (which is used as a reference point) is ok, but we need to change the size of the item.

   The layout is made with the following margins:

   - Left: 5 mm
   - Right: 5 mm
   - Top: 20 mm
   - Bottom: 10 mm

6. Having that as a reference, set the map item width to

   ```python
   @layout_pagewidth - 5 - 5
   ```
7. Set the item height to:

   ```python
   @layout_pageheight - 10 - 20
   ```

8. Change the preview to see the change happen

9. Before you go further with setting the rest of the items, notice that for this kind of map we will be using the margins sizes quite a lot. If, at some point, we need to change the size of the margins, all this expressions will be outdated. To prevent this, we can use variables.

   Variables are quite useful because you can used them in many different places (wherever you can use an expression), but, if you need to change its value, you will only need to do it in one place.

12. Open the Layout properties, at the bottom, set the following margins variables and values:

    - `margin_top` = 20
    - `margin_bottom` = 10
    - `margin_left` = 5
    - `margin_right` = 5

13. Go back to the map item properties and update the expression for width to:

    ```python
    @layout_pagewidth - @margin_left - @margin_right
    ```

    Set the height to:

    ```python
    @layout_pageheight - @margin_top - @margin_bottom
    ```

14. Addictionally, set the X and Y positions to depend on the margins variables too. Set `X` to `@margin_left`, and `Y` to `@margin_top`

15. Change the margins values, and then refresh to see the map moving to the right place.

16. Now we need to do the same for other items, starting by the legend. Set the reference point of the item to the bottom-right.
17. Set the X position to:

    ```
    @layout_pagewidth - @margin_right
    ```

18. Set the Y position to:

    ```
    @layout_pageheight
    ```
19. Set the height to:

    ```
    @margin_bottom
    ```

20. Let's do the same for all other items

    **Area label**
    Reference_point = bottom left corner
    X = @margin_left
    Y = @layout_pageheight
    width = 100
    Height = @margin_bottom

    **Credits label** (notice the angle)
    Reference_point = bottom right corner
    X = @layout_pagewidth - @margin_right
    Y = @layout_pageheight - @margin_bottom
    width = @margin_right
    Height = 267

    **Title label**
    Reference_point = top-center
    X = @layout_pagewidth / 2
    Y = 0
    width = 144
    Height = @margin_top

    **North Arrow**

    Reference_point = top-right
    X = @layout_pagewidth - @margin_right
    Y = 0
    width = 8.50
    Height = @margin_top

    **Legend**
    Reference_point = bottom-right
    X = @layout_pagewidth - @margin_right - 2
    Y = @layout_pageheight - @margin_bottom - 2
    width = 35
    Height = 16

    **Flag logo**
    Reference_point = top-left
    X = @margin_left
    Y = 0
    width = 22
    Height = @margin_top

Move the page preview and see all the maps taking places.

Note that, with all the work we have done, we can easily change the page size and margins, and get the layout to still work.

 ## exercise 5 - Make the page size change according to a fixed scale (optional).

In the last Notice that the scale of the map changes alot, the next example, will automatically change the size of the page to allow a fixed scale.

1. Duplicate map of the regions
2. On the layout properties, set a variable to a fixed scale 2000000
3. Edit the page properties and in the page size set the following expressions

   ```
   CASE
   WHEN min(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 210 AND
               max(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 297
   THEN 'A4'
   WHEN min(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 297 AND
               max(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 420
   THEN 'A3'
      WHEN min(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 420 AND
               max(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 594
   THEN 'A2'
      WHEN min(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 594 AND
               max(bounds_width(  @atlas_geometry ),bounds_height( @atlas_geometry)) / @scale * 304.8 <= 840
   THEN 'A1'
   ELSE 'A0'
  END
  ```

## Exercise 6 -