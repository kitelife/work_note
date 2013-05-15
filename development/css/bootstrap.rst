Bootstrap
=============

Global settings
------------------

**Requires HTML5 doctype**

Bootstrap makes use of certain HTML elements and CSS3 properties that require
the use of the HTML5 doctype. Including it at the beginning of all your
projects.::

    <!DOCTYPE html>
    <html lang="en">
    ...
    </html>

Default grid system
----------------------

The default Bootstrap grid system utilizes **12 columns**, making for a 940px
wide container without responsive features enabled. With the responsive CSS file
added, the grid adapts to be 720px and 1170px wide depending on your viewport.
Below 767px viewports, the columns become fluid and stack vertically.

**Offsetting columns**

**Nesting columns**

To nest your content with the default grid, add a new .row and set of `.span*`
columns within an existing `.span*` column. Nested rows should include a set of
columns that add up to the number of columns of its parent.


Fluid grid system
---------------------

The fluid grid system uses percents instead of pixels for column widths. It has
the same responsive capabilities as our fixed grid system, ensuring proper
proportions for key screen resolutions and devices.

**Basic fluid grid HTML**

Make any row "fluid" by changing `.row` ot `.row-fluid` . The column classes
stay the exact same, making it easy to flip between fixed and fluid grids.

**Fluid offsetting**

**Fluid nesting**

Fluid girds utilize nesting differently: each nested level of columns should add
up to 12 columns. This is because the fluid grid uses percentages, not pixels,
for setting widths.


Layouts
-----------

**Fixed layout**

Provides a common fixed-width (and optionally responsive) layout with only `<div
class="container">` required.

**Fluid layout**

Create a fluid, two-column page with `<div class="container-fluid">` --- great
for applications and docs.


Responsive design
---------------------

**Enabling responsive features**

Turn on responsive CSS in your project by including the proper meta tag and
additional stylesheet within the `<head>` of your document. If you've compiled
Bootstrap from the Customize page, you need only include the meta tag.::

    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="assets/css/bootstrap-responsive.css" rel="stylesheet">

**About responsive Bootstrap**

Media queries allow for custom CSS based on a number of conditions---ratios,
widths, display type, etc --- but usually focuses around `min-width` and
`max-width` .

- Modify the width of column in our grid
- Stack elements instead of float wherever necessary
- Resize headings and text to be more appropriate for devices

**Responsive utility classes**
