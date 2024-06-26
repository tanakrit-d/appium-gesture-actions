# Enhanced Mobile Scroll Library
 
## Work in Progress

The goal of this library is to provide more robust and useful scrolling functionality for Appium mobile automation.

We achieve this by dividing the viewport/screen into 'scrollable' regions, and then performing w3c actions within this space.
Using percentages of the viewport and element locations, we can avoid the standard approach of specifiying co-ordinates for each swipe/or scroll action.

## Code Example
```python
def swipe_element_into_view(self, locator_method: AppiumBy, locator_value: str):
    action = ActionChains(self.driver)
    action.w3c_actions = ActionBuilder(self.driver, mouse=PointerInput(Interaction.POINTER_TOUCHER, "touch"))
    element_x, element_y = self.retrieve_element_location(locator_method, locator_value)
    direction_x, direction_y = self.retrieve_relative_direction(element_x, element_y)

    if direction in [Direction.UP, Direction.DOWN]:
        distance_to_element = element_y - self.lower_bound
        actions_total = distance_to_element / self.scrollable_x
        actions_complete = distance_to_element // self.scrollable_x
        actions_partial = self.scrollable_x * (actions_total - int(actions_total))
        actions_complete = int(actions_complete)
        
        match direction:
            case Direction.UP:
                if actions_total > 1:
                    self.perform_navigation_full_y(action, self.upper_bound, self.lower_bound, actions_complete)
                if actions_partial > 50:
                    self.perform_navigation_partial_y(action, self.upper_bound, self.lower_bound)
            case Direction.DOWN:
                if actions_total > 1:
                    self.perform_navigation_full_y(action, self.lower_bound, self.upper_bound, actions_complete)
                if actions_partial > 50:
                    self.perform_navigation_partial_y(action, self.lower_bound, self.upper_bound)
```

## To-Do
- Write tests (lol)
- Publish as a library

## Understanding Element Location
Elements have two attributes: Position and Size.  
The position within the viewport is the top-left-point.

We can then use the element size to determine where it occupies relative to the view-port position.
![Element Diagram](resources/understanding_element_position-dimension.png)

```python
element_top_left_point      = element.location["x"], element.location["y"]
element_top_mid_point       = element.location["x"] + (element.size["width"] // 2), element.location["y"]
element_top_right_point     = element.location["x"] + element.size["width"], element.location["y"]

element_left_mid_point      = element.location["x"], element.location["y"] + (element.size["height"] // 2)
element_mid_point           = element.location["x"] + (element.size["width"] // 2), element.location["y"] + (element.size["height"] // 2)
element_right_mid_point     = element.location["x"] + element.size["width"], element.location["y"] + (element.size["height"] // 2)

element_bottom_left_point   = element.location["x"], element.location["y"] + element.size["height"]
element_bottom_mid_point    = element.location["x"] + (element.size["width"] // 2), element.location["y"] + element.size["height"]
element_bottom_right_point  = element.location["x"] + element.size["width"], element.location["y"] + element.size["height"]
```
Using the example element from the image, the above code would output the following:  
> Top-Left-Point:  (20, 20)  
> Top-Mid-Point:  (40, 20)  
> Top-Right-Point:  (60, 20)
> 
> Left-Mid-Point:  (20, 30)  
> Mid-Point:  (40, 30)  
> Right-Mid-Point:  (60, 30)
> 
> Bottom-Left-Point:  (20, 40)  
> Bottom-Mid-Point:  (40, 40)  
> Bottom-Right-Point:  (60, 40)

An MRE is available here: [demo/calc_coordinates.py](demo/calc_coordinates.py)
