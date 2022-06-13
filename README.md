# About
Tutorial of how to exclude items that do not have a cover from the Koha [Coverflow](https://github.com/bywatersolutions/koha-plugin-coverflow) plugin

#TL;DR
The Coverflow plugin created by ByWater Solutions is a great extension of the Koha ILS, but it uses Amazon to retrieve the cover images used in the plugin and if there are none available, it uses a placeholder 'No image available' cover.

We wanted to exclude these and wrote a line of JQuery to do so. However, when we placed the JQuery in OPACUserJS, nothing happened. Troubleshooting revealed that this was because the Coverflow plugin was being loaded after the OPACUserJS codeblock and therefore not being affected by our code.

Below are the steps taken to customize the plugin to exclude items without cover images.

## Step 1 - The code

The code we use is pretty straightforward. We select all images that have an `src` attribute that points to the 'No image available' placeholder cover and hide them.

`$("img[src='https://raw.githubusercontent.com/bywatersolutions/web-assets/master/NoImage.png']").hide();`

## Step 2 - Where to put the code

Normally we could include the above code in Home > Administration > System preferences > OPACUserJS, but because the Coverflow plugin is last thing loaded, we must include our code directly in the Coverflow code, specifically in the file `opacuserjs.tt`, which means we must find where the file lives.

If the Koha instance was installed via a Debian package, it should be located at: 
`/var/lib/koha/[instance name]/plugins/Koha/Plugin/Com/ByWaterSolutions/CoverFlow/opacuserjs.tt`

## Step 3 - Inserting the code

With opacuserjs.tt, find the reference to 'NoImage.png'. It should be around line 10.

Now simply create a new line and insert the code from **Step 1**.

```js
$(document).ready(function () {
    [% FOREACH m IN mapping %]
        [% IF m.sql_params.size > 0 %]
            $('[% m.selector %]').load("/api/v1/contrib/coverflow/reports/[% m.id | uri %]?[% m.sql_params.join('&') %]", function () {
        [% ELSE %]
            $('[% m.selector %]').load("/api/v1/contrib/coverflow/reports/[% m.id | uri %]", function () {
        [% END %]
                $('.koha-coverflow img').on("load", function(){
                if ( this.naturalHeight == 1 ) {
                    $(this).attr("src", [% IF (custom_image) %]"[% custom_image %]")[% ELSE %]"https://raw.githubusercontent.com/bywatersolutions/web-assets/master/NoImage.png")[% END%];
                    $("img[src='https://raw.githubusercontent.com/bywatersolutions/web-assets/master/NoImage.png']").hide(); // exclude items without cover images
                                    }
                });
            var opt = {
                'items': '.item',
                'minfactor': 15, // how much is the next item smaller than previous in pixels
                'distribution': 1.5, // how apart are the items (items become separated when this value is below 1)
                'scalethreshold': 0, // after how many items to start scaling
                'staticbelowthreshold': false, // if true when number of items is below "scalethreshold" - don't animate 
                'titleclass': 'itemTitle', // class name of the element containing the item title
                'selectedclass': 'selectedItem', // class name of the selected item
                'scrollactive': true, // scroll functionality switch
                'step': { // compressed items on the side are steps
                    'limit': 4, // how many steps should be shown on each side
                    'width': 10, // how wide is the visible section of the step in pixels
                    'scale': true // scale down steps
                }
            };
            $('[% m.selector %]').flipster({
                [% FOREACH option IN m.options.pairs %]
                    [% option.key %]: '[% option.value %]',
                [% END %]
            });
        });
    [% END %]
});
```

Note that this code will likely be overwritten by any future updates to the Coverflow plugin and will need to be reinserted.
