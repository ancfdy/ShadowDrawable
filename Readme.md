通过一行代码即可实现阴影效果

    ShadowDrawable.setShadowDrawable(textView1, Color.parseColor("#3D5AFE"), dpToPx(8),
        Color.parseColor("#66000000"), dpToPx(10), 0, 0);
Android开发中阴影效果的实现
背景
随着这几年UI风格的不断升级，阴影已经成了很多APP设计中的不可或缺的元素。但Android在这方面却没有比较好的实现方式。

这里有总结的一篇关于Android阴影效果的文章，比较全面，值得一看。聊聊 Material Design 里，阴影的那些事儿！

上面这篇文章对Android中各版本的阴影实现都进行了说明，这里就不再细说了。虽然提供的方式很多，但是却有很大的局限性，具体表现在以下两方面：

无法改变阴影的颜色；
存在兼容性问题；
虽然也可以按照作者提供的方式，使用Fab或CardView实现阴影的原理来实现，但相对比较麻烦，这里我们提供一个简单的实现方案。

实现思想
为View添加阴影，其实就是为View提供一个有阴影的背景而已，所以有2中实现方式：

重写View的onDraw()方法；
自定义Drawable；
第一种明显不合理，我们不可能重写每个需要设置阴影的View的onDraw()，所以这里选择自定义Drwable（通过设置Paint的ShadowLayer）来实现。需要注意的是：这种方式实现的阴影，其目标View需要关闭硬件加速。

view.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
需求点
可设置阴影颜色，圆角，面积，偏移量；
可设置View的背景形状，颜色，圆角；
实现
源码地址：ShadowDrawable

public class ShadowDrawable extends Drawable {

	private Paint mPaint;
	private int mShadowRadius;  // 阴影圆角
	private int mShape;         // 背景形状
	private int mShapeRadius;   // 背景圆角
	private int mOffsetX;       // 阴影的水平偏移量
	private int mOffsetY;       // 阴影的垂直偏移量
	private int mBgColor[];     // 背景颜色
	private RectF mRect;

	public final static int SHAPE_ROUND = 1;    // 表示圆角矩形
	public final static int SHAPE_CIRCLE = 2;   // 表示圆

	private ShadowDrawable(int shape, int[] bgColor, int shapeRadius, int shadowColor, int shadowRadius, int offsetX, int offsetY) {
		this.mShape = shape;
		this.mBgColor = bgColor;
		this.mShapeRadius = shapeRadius;
		this.mShadowRadius = shadowRadius;
		this.mOffsetX = offsetX;
		this.mOffsetY = offsetY;
		mPaint = new Paint();
		mPaint.setColor(Color.TRANSPARENT);
		mPaint.setAntiAlias(true);
		mPaint.setShadowLayer(shadowRadius, offsetX, offsetY, shadowColor);
	    mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_ATOP));
	}

	@Override
	public void setBounds(int left, int top, int right, int bottom) {
		super.setBounds(left, top, right, bottom);
		mRect = new RectF(left + mShadowRadius - mOffsetX, top + mShadowRadius - mOffsetY, right - mShadowRadius - mOffsetX,
				bottom - mShadowRadius - mOffsetY);
	}

	@Override
	public void draw(@NonNull Canvas canvas) {
		if (mShape == SHAPE_ROUND) {
			canvas.drawRoundRect(mRect, mShapeRadius, mShapeRadius, mPaint);
			Paint newPaint = new Paint();

			if (mBgColor != null) {
				if (mBgColor.length == 1) {
					newPaint.setColor(mBgColor[0]);
				} else {
					newPaint.setShader(new LinearGradient(mRect.left, mRect.height() / 2, mRect.right, mRect.height() / 2, mBgColor,
							null, Shader.TileMode.CLAMP));
				}
			}
			newPaint.setAntiAlias(true);
			canvas.drawRoundRect(mRect, mShapeRadius, mShapeRadius, newPaint);
		} else {
			canvas.drawCircle(mRect.centerX(), mRect.centerY(), Math.min(mRect.width(), mRect.height())/ 2, mPaint);
		}
	}

	@Override
	public void setAlpha(int alpha) {
		mPaint.setAlpha(alpha);
	}

	@Override
	public void setColorFilter(@Nullable ColorFilter colorFilter) {
		mPaint.setColorFilter(colorFilter);
	}

	@Override
	public int getOpacity() {
		return PixelFormat.TRANSLUCENT;
	}
}
由于提供的属性比较多，为了便于使用，提供了Builder的链式创建方式，同时提供了常用的几个static方法，设置阴影只需一行代码即可，具体查看ShadowDrawable.java。

public static void setShadowDrawable(View view, int shapeRadius, int shadowColor, int shadowRadius, int offsetX, int offsetY) {
	ShadowDrawable drawable = new ShadowDrawable.Builder()
			.setShapeRadius(shapeRadius)
			.setShadowColor(shadowColor)
			.setShadowRadius(shadowRadius)
			.setOffsetX(offsetX)
			.setOffsetY(offsetY)
			.builder();
	view.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
	ViewCompat.setBackground(view, drawable);
}
实例效果
image
注意点
设置阴影的颜色时，需要带有透明度(即"#XXXXXXXX"的形式，如50%的黑色["#80000000"])，而不能使用纯色；
上面提供的这种实现方式，阴影部分总是作为View的一部分而存在的,所以在使用时，需要为阴影留出相对应的padding, 才会让阴影显示出来;
总结
由于Android系统并没有提供完美的解决方案，即便使用5.0以上提供的elevation或translationZ属性，对于大面积的阴影，看起来也比较生硬， 所以在Android开发中，对于阴影的处理，还是应该分情况来对待，大面积的阴影强烈推荐使用9Patch图来解决，对于小控件的阴影，可使用以上的方案。
