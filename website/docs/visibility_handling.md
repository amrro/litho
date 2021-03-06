---
id: visibility-handling
title: Visibility Handling
---

### Types of Visibility Range

The framework currently supports six types of Visibility Event:

- [Visible Event](/javadoc/com/facebook/litho/VisibleEvent.html): this event is triggered when at least one pixel of the Component is visible.
- [Invisible Event](/javadoc/com/facebook/litho/InvisibleEvent.html): this event is triggered when the Component no longer has any pixels on the screen.
- [Focused Visible Event](/javadoc/com/facebook/litho/FocusedVisibleEvent.html): this event is triggered when either the Component occupies at least half of the viewport, or, if the Component is smaller than half the viewport, when it is fully visible.
- [Unfocused Visible Event](/javadoc/com/facebook/litho/UnfocusedVisibleEvent.html): this event is triggered when the Component is no longer focused, i.e. it is not fully visible and does not occupy at least half the viewport.
- [Full Impression Visible Event](/javadoc/com/facebook/litho/FullImpressionVisibleEvent.html): this event is triggered when the entire Component has passed through the viewport at some point.
- [VisibilityChangedEvent](/javadoc/com/facebook/litho/VisibilityChangedEvent.html): this event is triggered when the visibility of the Component on the screen changes.

### Usage

:::caution IMPORTANT
Visibility ranges require [incremental mount](inc-mount#manual-incremental-mount) to be enabled on the relevant Component.
:::

To register visibility event handlers for a component you can follow the same [steps](events-overview) as for setting any other event handler. 

Here is an example:

```java
@LayoutSpec
class MyLayoutSpec {

  @OnCreateLayout
  static Component onCreateLayout(ComponentContext c) {

    return Column.create(c)
        .alignItems(YogaAlign.STRETCH)
        .child(Text.create(c)
            .text("This is MY layout spec")
            .visibleHandler(MyLayout.onTitleVisible(c))
            .invisibleHandler(MyLayout.onTitleInvisible(c))
            .focusedHandler(MyLayout.onComponentFocused(c, "someStringParam"))
            .fullImpressionHandler(MyLayout.onComponentFullImpression(c)))
            .visibilityChangedHandler(MyLayout.onComponentVisibilityChanged(c))
        .build();
  }

  @OnEvent(VisibleEvent.class)
  static void onTitleVisible(ComponentContext c) {
    Log.d("VisibilityRanges", "The title entered the Visible Range");
  }

  @OnEvent(InvisibleEvent.class)
  static void onTitleInvisible(ComponentContext c) {
    Log.d("VisibilityRanges", "The title is no longer visible");
  }

  @OnEvent(FocusedVisibleEvent.class)
  static void onComponentFocused(
      ComponentContext c,
      @Param String stringParam) {
    Log.d(
        "VisibilityRanges",
        "The component is focused with param: " + stringParam);
  }

  @OnEvent(FullImpressionVisibleEvent.class)
  static void onComponentFullImpression(ComponentContext c) {
    Log.d("VisibilityRanges", "The component has logged a full impression");
  }

  @OnEvent(VisibilityChangedEvent.class)
  static void onComponentVisibilityChanged(
      ComponentContext c,
      @FromEvent int visibleHeight,
      @FromEvent int visibleWidth,
      @FromEvent float percentVisibleHeight,
      @FromEvent float percentVisibleWidth) {
      Log.d("VisibilityRanges", "The component visible size is " + visibleHeight + "h" + visibleWidth + "w");
  }

}
```

:::info
VisibilityChangedEvents should be used with particular care since they will be dispatched on every frame while scrolling. No heavy work should be done inside the VisibilityChangedEvents handlers. Visible, Invisible, Focused, Unfocused and Full Impression events are the recommended over VisibilityChanged events whenever possible.
:::

### Custom visibility percentage
By default, a visibility event is triggered when a Component is fully visible. In some cases you may want to listen to custom visibility events and perform an action when the Component is only partially visible.
You can specify a ratio of the Component width or height for when the visibility event is dispatched by using `visibleHeightRatio` and `visibleWidthRatio` props when specifying a visibility handler.

```java
@OnCreateLayout
  static Component onCreateLayout(ComponentContext c) {

    return Column.create(c)
        .alignItems(YogaAlign.STRETCH)
        .child(Text.create(c)
            .text("This is MY layout spec")
            .visibleHandler(MyLayout.onTitleVisible(c))
            .visibleHeightRatio(0.8f)
            .visibleWidthRatio(0.1f)
        .build();
}
```
For the example above, a VisibilityEvent is dispatched when at least 80% of the Component's height and 10% of the Component's width is visible.
When the Component's visible percentage changes to less than 80% of total height, an invisible event will be dispatched.
If not specified, the default width or height ratio is 1f.

### Changing LithoView visibility
There are cases when you need to trigger visibility events on the LithoView components because the UI visibility changed, but the UI did not receive any callback to inform it of this change. An example is when a new activity is added to the back stack, covering the UI. In such cases you can call `setVisibilityHint` on the `LithoView` to tell the UI whether it is visible or not. You may want to do this when `Fragment#setUserVisibleHint` or `onResume/onPause` are called.
An example is when `Fragment#setUserVisibleHint` or `onResume/onPaused` are called.

Example usage:
```java
LithoView.setVisibilityHint(true); // This will dispatch visible/focused events as necessary on all components inside this LithoView
LithoView.setVisibilityHint(false); // This will dispatch invisible/unfocused events as necessary on all components inside this LithoView
```

### Troubleshooting
If you are not seeing your visibility event fired when you expect it to be, you can take the following steps: 
1. Verify that incremental mount is enabled for your Component. It is enabled by default, but if you turned it off, then visibility events will not be fired. 
2. Verify that you actually set the event that you defined in your spec on your Component (i.e. by calling `visibleHandler(MyLayout.onTitleVisible(c))` or similar. 
3. Visibility handlers are fired in [`MountState.processVisibilityOutputs()`](https://github.com/facebook/litho/blob/master/litho-core/src/main/java/com/facebook/litho/MountState.java#L489:L657). You can step through this method and see why the event that you expect to be fired is not being fired. 
