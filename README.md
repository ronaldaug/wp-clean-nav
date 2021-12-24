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

## 1. Build wordpress nav array data

```php
   /**
   * Get Menu Array
   *
   * @param string $name
   * @return array
   */
function get_menu_array(string $name) : array {

      function buildTree(array &$flatNav, $parentId = 0) {
          $branch = [];

          foreach ($flatNav as &$navItem) {
            if($navItem->menu_item_parent == $parentId) {
              $children = buildTree($flatNav, $navItem->ID);
              if($children) {
                $navItem->children = $children;
              }

              $branch[$navItem->menu_order] = $navItem;
              unset($navItem);
            }
          }

          return $branch;
      }

$locations = get_nav_menu_locations();
$flatMainNav = wp_get_nav_menu_items($locations[$name]);
return buildTree($flatMainNav);

}
```

### 2. Generate Menu UI from above `get_menu_array` array

```php
if(!function_exists('generate_menu')){
        /**
        * Generate Menu
        *
        * @param array $menus
        * @param string $class
        * @return void
        */
    function generate_menu(array $menus, string $class) : string {
        $sub_menu = '<ul class="'.$class.'">';
        foreach($menus as $menu){
            $children = $menu->children;
            $sub_menu .= empty($children) ? menu_li($menu) : menu_li($menu,$children);
        }
        return $sub_menu . '</ul>';
    }
}

if (!function_exists('menu_li')) {
    /**
        * Get single menu li
        *
        * @param object $menu
        * @param array|null $children
        * @return void
        */
    function menu_li(object $menu, array $children = null) : string
    {
        $active = $menu->object_id == get_queried_object_id() ? 'class="active"' : '';
        if (empty($children)) {
            return '<li '.$active.'><a href="'.$menu->url.'">'.$menu->title.'</a></li>';
        }
        return '<li '.$active.'><a href="'.$menu->url.'">'.$menu->title.'</a>'. generate_menu($children, 'sub-menu') .'</li>';
    }
}

$menus = get_menu_array('primary');

echo generate_menu($menus,'parent-menu');
```
