# WP clean UI navbar

- No extra classes from Wordpress
- Has active class
- Build with recursive function and it can generate multiple sub-menus ( 2nd level, 3rd level, 4th level ..etc)

### Example output

```html
<ul class="parent-menu">
    <li><a href="/">Item1</a>
    
        <!-- sub menu -->
        <ul class="sub-menu">
            <li><a href="/">dropdown 1</a></li>
            <li><a href="/">dropdown 2</a>
            

                <!-- sub menu -->
                <ul class="sub-menu">
                    <li><a href="/">dropdown 3</a></li>
                    <li><a href="/">dropdown 4</a></li>
                </ul>
                
            </li>
        </ul>
        
    </li>
    <li><a href="/">Item2</a></li>
    <li><a href="/">Item3</a></li>
</ul>
```



-------------------------------------
### Usage 

1. [Create `clean-menu` file in your theme like this `classes/clean-menu.php`](#createfile)

2. [Add below code in functions.php](#functions)

3. [Call it somewhere in your wordpress template](#callit)


### 1. Create `clean-menu.php` file in your theme with below code ( CLEAN_Menu ).

e.g `classes/clean-menu.php` <a name="createfile"></a>

```php
<?php

    class CLEAN_menu
    {
        /**
         *  Get Menu List by menu name
         *
         * @param string $name
         * @return array
         */
        public static function get_data(string $name) : array
        {
            function buildTree(array &$flatNav, $parentId = 0)
            {
                $branch = [];
        
                foreach ($flatNav as &$navItem) {
                    if ($navItem->menu_item_parent == $parentId) {
                        $children = buildTree($flatNav, $navItem->ID);
                        if ($children) {
                            $navItem->children = $children;
                        }
        
                        $branch[$navItem->menu_order] = $navItem;
                        unset($navItem);
                    }
                }
        
                return $branch;
            }

            // get navs
            $locations = get_nav_menu_locations();

            // get menu items by menu name
            $flatMainNav = wp_get_nav_menu_items($locations[$name]);
            return buildTree($flatMainNav);
        }


        /**
        * Generate Menu
        *
        * @param array $menus
        * @param string $class
        * @return void
        */
        public static function generate_menu(array $menus, string $class) : string
        {
            $sub_menu = '<ul class="'.$class.'">';
            foreach ($menus as $menu) {
                $children = $menu->children;
                $sub_menu .= empty($children) ? self::menu_li($menu) : self::menu_li($menu, $children);
            }
            return $sub_menu . '</ul>';
        }


        /**
		* Get single menu li
		*
		* @param object $menu
		* @param array|null $children
		* @return void
		*/
        protected static function menu_li(object $menu, array $children = null) : string
        {
            $active = $menu->object_id == get_queried_object_id() ? 'class="active"' : '';
            if (empty($children)) {
                return '<li '.$active.'><a href="'.$menu->url.'">'.$menu->title.'</a></li>';
            }
            return '<li '.$active.'><a href="'.$menu->url.'">'.$menu->title.'</a>'. self::generate_menu($children, 'sub-menu') .'</li>';
        }
    }

```
--------------


### 2. Add below code in functions.php <a name="functions"></a>

```php
/* -------------------------------------------
Register Menu
---------------------------------------------- */
function register_clean_menu(){
	require_once get_template_directory() . '/classes/clean-menu.php';
}
add_action( 'after_setup_theme', 'register_clean_menu' );

register_nav_menus( array(
    'primary' => __( 'Top Menu', 'kthm' ),
    'secondary' => __( 'Other pages Menu', 'kthm' ),
) );



if (!function_exists('clean_menus')) {

/**
 *  Get Menu List by menu name
 *
 * @param string $name
 * @return mixed
 */
function clean_menus(string $name){
    // get menu array
    $menus = CLEAN_menu::get_data($name);

    // generate menu UI
    return CLEAN_menu::generate_menu($menus,'parent-menu');
}

}

```

--------------


### 3. Call it somewhere in your wordpress template <a name="callit"></a>
```php
<?= clean_menus(); ?>
```
