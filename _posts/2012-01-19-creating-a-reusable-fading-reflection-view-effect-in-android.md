---
layout: post
title: Creating a reusable "fading reflection" view effect in Android
categories: android
tags: android java layout ui views
status: publish
type: post
published: true
comments: true
---

Recently I needed to create various Android UIs which provides a "fading reflection" effect for certain screen elements. I happened upon <a title="Android, Reflections with Bitmaps" href="http://www.inter-fuser.com/2009/12/android-reflections-with-bitmaps.html" target="_blank">Neil Davies excellent post</a> on creating reflections of images in an Activity in order to re-create something similar to Apple's coverflow effect, and it set me thinking about what could be achieved using this technique.

![Apple's Coverflow](/assets/images/coverflow.jpg)Apple's "Coverflow" provides a nice reflection effect


Really what I wanted was a way to take the approach outlined by Neil and easily apply it to potentially any View or collection of Views in a given UI without needing to write the same boilerplate in the Activity every time.

For example, lets say I have a basic layout for showing progress to which I want to add a "reflection" effect:
<table>
  <tbody>
    <tr>
      <td>
        <img src="/assets/images/progress_bar.png" alt="Without" />
        Original
      </td>
      <td>
        <img src="/assets/images/progress_bar_with_reflection.png" alt="Original with added reflection" />
        Original with added reflection
      </td>
    </tr>
  </tbody>
</table>
You can see that there are a couple of View objects in there (a `ProgressBar` and a `TextView`) which we'd like to apply the reflection effect to, rather than a simple image. Moreover, it would be nice to be able to add/remove UI elements at runtime and have them included in the effect as required.

# Introducing ReflectingLayout

![Reflecting layout diagram](/assets/images/reflecting_layout_diagram.png)

To implement this reusable reflection effect, I created a new custom View based on LinearLayout named "ReflectingLayout". ReflectingLayout will simply apply a reflection effect to child views declared within it using the remaining space not used below those child views.

As the diagram to the right illustrates, the idea is that the layout params of the ReflectingLayout and the contained views are such that an empty region is left within the ReflectingLayout below the child views which can be used to draw a reflection. You can see that if the ReflectingLayout's height is reduced, the area available for the reflection is also reduced.

This approach allows you to potentially insert any combination of child views, and provided you've allowed enough space below them within the ReflectedLayout container, you can achieve a reflection effect on anything.

# The Code
As well as working through the approach here, I've made the source of ReflectingLayout <a title="GitHub" href="https://gist.github.com/1640793">available on GitHub</a> if you want to reuse it, make improvements etc.

This is the declaration of ReflectingLayout:

{% highlight java %}

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.LinearGradient;
import android.graphics.Matrix;
import android.graphics.Paint;
import android.graphics.PorterDuff.Mode;
import android.graphics.PorterDuffXfermode;
import android.graphics.Shader.TileMode;
import android.util.AttributeSet;
import android.widget.LinearLayout;

/**
 * A general purpose Layout which simply renders a reflection of its contained
 * child Views in the remaining space below them within the bounds of this control.
 * For {@link ReflectingLayout} to work properly, it must be setup to provide
 * sufficient empty space below its children.
 *
 * Copyright Tom
 */
public class ReflectingLayout extends LinearLayout {

  /** The maximum ratio of the height of the reflection to the source image. */
  private static final float MAX_REFLECTION_RATIO = 0.9F;

  /** The {@link Paint} object we'll use to create the reflection. */
  private Paint paint;

  private Matrix vFlipMatrix;

  /**
   * Instantiates a new reflecting layout.
   *
   * @param context the context
   * @param attrs the attrs
   */
  public ReflectingLayout(Context context, AttributeSet attrs) {
    super(context, attrs);
    init();
  }

  /**
   * Instantiates a new reflecting layout.
   *
   * @param context the context
   */
  public ReflectingLayout(Context context) {
    super(context);
    init();
  }

  /**
   * Initialises the layout.
   */
  private void init() {
    // Ensures that we redraw when our children are redrawn.
    setAddStatesFromChildren(true);

    // Important to ensure onDraw gets called.
    setWillNotDraw(false);
    setDrawingCacheEnabled(true);

    // Create the paint object which we'll use to create the reflection gradient
    paint = new Paint();
    paint.setXfermode(new PorterDuffXfermode(Mode.DST_IN));

    // Create a matrix which can flip images vertically
    vFlipMatrix = new Matrix();
    vFlipMatrix.preScale(1, -1);
  }

  /**
   * {@inheritDoc}
   * @see android.view.View#onDraw(android.graphics.Canvas)
   */
  @Override
  protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    // Only actually do anything if there is space to actually draw a reflection
    if (getReflectionHeight() > 0) {

      // Create a bitmap to hold the drawing of child views and pass this to a temp canvas
      Bitmap sourceBitmap = Bitmap.createBitmap(getMeasuredWidth(), getMeasuredHeight(),
          Bitmap.Config.ARGB_8888);
      Canvas tempCanvas = new Canvas(sourceBitmap);

      // Draw the content of this layout onto our temporary canvas.
      super.dispatchDraw(tempCanvas);

      // Calculate the height of the reflection and the bottom position of child views.
      int reflectionHeight = getReflectionHeight();
      int childBottom = getMaxChildBottom();

      // Create a new bitmap from the source image which has been vertically flipped and
      // only includes the region occupied child views.
      Bitmap flippedBitmap = Bitmap.createBitmap(
          sourceBitmap,
          0,
          0,
          sourceBitmap.getWidth(),
          childBottom,
          vFlipMatrix,
          false);

      // Create a bitmap to hold just the reflection
      Bitmap fadedBitmap = Bitmap.createBitmap(
          getMeasuredWidth(), reflectionHeight, Bitmap.Config.ARGB_8888);

      Canvas fadeCanvas = new Canvas(fadedBitmap);
      fadeCanvas.drawBitmap(flippedBitmap, 0, 0, null);

      LinearGradient gradient = new LinearGradient(0, 0, 0, reflectionHeight,
          0x30FFFFFF, 0x00FFFFFF, TileMode.CLAMP);
      paint.setShader(gradient);

      // Now use some clever PorterDuff shading to get the fading effect.
      fadeCanvas.drawRect(0, 0, getMeasuredWidth(), reflectionHeight, paint);

      // Draw our image onto the canvas
      canvas.drawBitmap(fadedBitmap, 0, childBottom, null);

    }
  }

  /**
   * Finds the bottom of the lowest view contained by this layout.
   *
   * @return the bottom of the lowest view
   */
  private int getMaxChildBottom() {
    int maxBottom = 0;
    for (int i = 0; i < getChildCount(); i++) {
      int bottom = getChildAt(i).getBottom();
      if (bottom > maxBottom) maxBottom = bottom;
    }
    return maxBottom;
  }

  /**
   * Gets the highest top edge of all contained views.
   *
   * @return the min child top
   */
  private int getMinChildTop() {
    int minTop = Integer.MAX_VALUE;
    for (int i = 0; i < getChildCount(); i++) {
      int top = getChildAt(i).getTop();
      if (top < minTop) minTop = top;
    }
    return minTop;
  }

  /**
   * Gets the height of the space covered by all children.
   *
   * @return the total child height
   */
  private int getTotalChildHeight() {
    // The max value of any child's "bottom" minus the minimum of any "top"
    return getMaxChildBottom() - getMinChildTop();
  }

  /**
   * Gets the height of the reflection to be drawn.
   *
   * @return the reflection height
   */
  private int getReflectionHeight() {
    return (int) Math.min(
      getMeasuredHeight() - getMaxChildBottom(),
      getTotalChildHeight() * MAX_REFLECTION_RATIO);
  }

{% endhighlight %}

So what's going on here? First let's take a look at the init() method, used by both constructors to initialise the ReflectingLayout:

{% highlight java %}
  /**
   * Initialises the layout.
   */
  private void init() {
    // Ensures that we redraw when our children are redrawn.
    setAddStatesFromChildren(true);

    // Important to ensure onDraw gets called.
    setWillNotDraw(false);
    setDrawingCacheEnabled(true);

    // Create the paint object which we'll use to create the reflection gradient
    paint = new Paint();
    paint.setXfermode(new PorterDuffXfermode(Mode.DST_IN));

    // Create a matrix which can flip images vertically
    vFlipMatrix = new Matrix();
    vFlipMatrix.preScale(1, -1);
  }
{% endhighlight %}

The first three set...() calls ensure that the `ReflectingLayout` pays attention to its children and makes use of the drawing cache to ensure we only do a full redraw when necessary. We also create `Paint` and `Matrix` objects for use later on in `onDraw()`. This one-off instantiation is a good idea to ensure we don't waste memory and do more garbage collection than necessary later on by repeatedly creating these objects in `onDraw()` which will be called a lot. The Paint is configured with the Porter-Duff transfer mode, this is what allows us to use a simple color gradient to vertically fade out images. The `vFlipMatrix` is simply configured to flip an image vertically to make it appear upside down. The `onDraw()` method is what the Android framework will call on a given View object when it want that View (and it's children) to draw itself onto the given canvas. `onDraw()` methods tend to get called a lot in quick succession as a UI updates, so must be fast. Here's `ReflectingLayout`'s implementation:

{% highlight java %}
protected void onDraw(Canvas canvas) {
  super.onDraw(canvas);
  // Only actually do anything if there is space to actually draw a reflection
  if (getReflectionHeight() > 0) {

      // Create a bitmap to hold the drawing of child views and pass this to a temp canvas
      Bitmap sourceBitmap = Bitmap.createBitmap(
          getMeasuredWidth(), getMeasuredHeight(), Bitmap.Config.ARGB_8888);
      Canvas tempCanvas = new Canvas(sourceBitmap);

      // Draw the content of this layout onto our temporary canvas.
      super.dispatchDraw(tempCanvas);

      // Calculate the height of the reflection and the bottom position of child views.
      int reflectionHeight = getReflectionHeight();
      int childBottom = getMaxChildBottom();

      // Create a new bitmap from the source image which has been vertically flipped and
      // only includes the region occupied child views.
      Bitmap flippedBitmap = Bitmap.createBitmap(
          sourceBitmap,
          0,
          0,
          sourceBitmap.getWidth(),
          childBottom,
          vFlipMatrix,
          false);

      // Create a bitmap to hold just the reflection
      Bitmap fadedBitmap = Bitmap.createBitmap(
          getMeasuredWidth(), reflectionHeight, Bitmap.Config.ARGB_8888);

      Canvas fadeCanvas = new Canvas(fadedBitmap);
      fadeCanvas.drawBitmap(flippedBitmap, 0, 0, null);

      LinearGradient gradient = new LinearGradient(0, 0, 0, reflectionHeight,
          0x30FFFFFF, 0x00FFFFFF, TileMode.CLAMP);
      paint.setShader(gradient);

      // Now use some clever PorterDuff shading to get the fading effect.
      fadeCanvas.drawRect(0, 0, getMeasuredWidth(), reflectionHeight, paint);

      // Draw our image onto the canvas
      canvas.drawBitmap(fadedBitmap, 0, childBottom, null);
    }
{% endhighlight %}

Hopefully the code comments are fairly self explanatory, but the general approach is as follows:

* It's important to call `super.onDraw(canvas)`. This will ensure that the child views are drawn in the normal way onto the main canvas. Otherwise we'd appear to have a reflection of some non-existent objects.
* We then get the child views to draw themselves again into a `Bitmap` object via a temporary `Canvas` we create. This then gives us a Bitmap we can manipulate to achieve the reflection effect.
* The `Bitmap` is used to created a second version which has been cropped at the "childBottom" (the y position of the bottom most edge of any child View, in our case the `TextView`) and then flipped vertically. This then forms the basis for our reflection image.
* We then configure our Porter-Duff-enabled `Paint` object with a vertical gradient covering the full height of the reflection image, so that the fading out of the reflection is correctly calibrated.
* The `Paint` is then used to "draw" a rectangle over base reflection image, having the effect of fading it out.
* Finally the resultant Bitmap is drawn onto the main canvas starting just below the bottom of the child Views.

# A couple of final points
I have noticed that certain changes made at runtime to child Views of `ReflectingLayout` sometimes don't result in the `ReflectingLayout` getting redrawn. I'm not 100% sure why this is but I believe it's related to Android's view optimisations trying to prevent redraws whenever possible. If Android thinks it can get away with only redrawing a given child of ReflectingLayout in response to a change to that just that view (e.g. a change of text color, rather than a change in layout) then `onDraw()` on `ReflectingLayout` does not get called and the reflection does not get updated.

This is a bit annoying. I had hoped there would be come mechanism to "hook in" to child redraw events from ReflectingLayout but have not managed to find anything. If anyone can suggest a suitable approach please let me know. For now, I have resorted to calling `invalidate()` on the ReflectingLayout whenever updating one of its children to ensure the reflection is refreshed.

Happy reflecting!
