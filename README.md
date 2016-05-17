# ViewBadgeDemo
ViewBadge demo for personal use

这是沿用https://github.com/jgilfelt/android-viewbadger   这篇github所实现的一个中文demo。

ViewBadge用于在app内实现消息提醒，添加注解数目的一个工具包。主要的实现过程为（嵌入到自己的project）:

1.将lib目录下android-viewbadge.jar包拷贝进libs目录下，并add as library

2.在对用的代码位置初始化BadgeView,并将需要实现功能的控件放进badge中，这里给出一个初始的实例：
           View target = findViewById(R.id.target_view);
           BadgeView badge = new BadgeView(this, target);
           badge.setText("1");
           badge.show();

3.其他具体功能后续介绍



功能模块：
由于BadgeView是继承TextView，所以TextView所拥有的功能，BadgeView都拥有

1.默认格式
           View target = findViewById(R.id.target_view);
           BadgeView badge = new BadgeView(this, target);
           badge.setText("1");
           badge.show();
2.设置所处位置
           badge1.setBadgePosition(BadgeView.POSITION_CENTER);

         然后附下Badge中提供的位置参数
         
            public static final int POSITION_TOP_LEFT = 1;//顶部左边
            public static final int POSITION_TOP_RIGHT = 2;//顶部右边
            public static final int POSITION_BOTTOM_LEFT = 3;//底部左边
            public static final int POSITION_BOTTOM_RIGHT = 4;//底部右边
            public static final int POSITION_CENTER = 5;//中间
3.设置badge的背景颜色/文字大小
           badge2.setTextColor(Color.BLUE);
           badge2.setBadgeBackgroundColor(Color.YELLOW);
           badge2.setTextSize(12);
4.更改badge的距离
           badge4.setBadgeMargin(15, 10);//前者为horizatal。后者为vertical方向
           
5.更改动画效果
          TranslateAnimation anim = new TranslateAnimation(-100, 0, 0, 0);
                anim.setInterpolator(new BounceInterpolator());
                anim.setDuration(1000);
                badge4.toggle(anim, null);//ain为进入时的动画，后为消失时的动画
                
                
                
                
其他的一些功能请看MainActivity的注解.附上BadgeView的源码


     package viewbadge.langyi.viewbadgedemo.badgeview;

import android.content.Context;
import android.content.res.Resources;
import android.graphics.Color;
import android.graphics.Typeface;
import android.graphics.drawable.ShapeDrawable;
import android.graphics.drawable.shapes.RoundRectShape;
import android.util.AttributeSet;
import android.util.TypedValue;
import android.view.Gravity;
import android.view.View;
import android.view.ViewGroup;
import android.view.ViewGroup.LayoutParams;
import android.view.ViewParent;
import android.view.animation.AccelerateInterpolator;
import android.view.animation.AlphaAnimation;
import android.view.animation.Animation;
import android.view.animation.DecelerateInterpolator;
import android.widget.FrameLayout;
import android.widget.TabWidget;
import android.widget.TextView;

/**
 * A simple text label view that can be applied as a "badge" to any given {@link View}.
 * This class is intended to be instantiated at runtime rather than included in XML layouts.
 * 一个实例的text 标签，用来作为一个“标记”给其他控件
 * 这个类用于在运行时生成而非在xml布局中写出
 * @author Jeff Gilfelt
 */
public class BadgeView extends TextView {

	//位置参数
	public static final int POSITION_TOP_LEFT = 1;
	public static final int POSITION_TOP_RIGHT = 2;
	public static final int POSITION_BOTTOM_LEFT = 3;
	public static final int POSITION_BOTTOM_RIGHT = 4;
	public static final int POSITION_CENTER = 5;

	//默认距离 位置 颜色值
	private static final int DEFAULT_MARGIN_DIP = 5;
	private static final int DEFAULT_LR_PADDING_DIP = 5;
	private static final int DEFAULT_CORNER_RADIUS_DIP = 8;
	private static final int DEFAULT_POSITION = POSITION_TOP_RIGHT;
	private static final int DEFAULT_BADGE_COLOR = Color.parseColor("#CCFF0000"); //Color.RED;
	private static final int DEFAULT_TEXT_COLOR = Color.WHITE;

	//默认弹进，弹出动画
	private static Animation fadeIn;
	private static Animation fadeOut;
	
	private Context context;
	private View target;
	
	private int badgePosition;
	private int badgeMarginH;
	private int badgeMarginV;
	private int badgeColor;
	
	private boolean isShown;
	
	private ShapeDrawable badgeBg;
	
	private int targetTabIndex;
	
	public BadgeView(Context context) {
		this(context, (AttributeSet) null, android.R.attr.textViewStyle);
	}
	
	public BadgeView(Context context, AttributeSet attrs) {
		 this(context, attrs, android.R.attr.textViewStyle);
	}
	
	/**
     * Constructor -
     * 常用的构造方法
     * create a new BadgeView instance attached to a target {@link View}.
     *
     * @param context context for this view.
     * @param target the View to attach the badge to.
     */
	public BadgeView(Context context, View target) {
		 this(context, null, android.R.attr.textViewStyle, target, 0);
	}
	
	/**
     * Constructor -
     * 
     * create a new BadgeView instance attached to a target {@link TabWidget}
     * tab at a given index.
     *
     * @param context context for this view.
     * @param target the TabWidget to attach the badge to.
     * @param index the position of the tab within the target.
     */
	public BadgeView(Context context, TabWidget target, int index) {
		this(context, null, android.R.attr.textViewStyle, target, index);
	}
	
	public BadgeView(Context context, AttributeSet attrs, int defStyle) {
		this(context, attrs, defStyle, null, 0);
	}
	
	public BadgeView(Context context, AttributeSet attrs, int defStyle, View target, int tabIndex) {
		super(context, attrs, defStyle);
		init(context, target, tabIndex);
	}

	private void init(Context context, View target, int tabIndex) {
		
		this.context = context;
		this.target = target;
		this.targetTabIndex = tabIndex;
		
		// apply defaults
		badgePosition = DEFAULT_POSITION;
		badgeMarginH = dipToPixels(DEFAULT_MARGIN_DIP);
		badgeMarginV = badgeMarginH;
		badgeColor = DEFAULT_BADGE_COLOR;
		
		setTypeface(Typeface.DEFAULT_BOLD);
		int paddingPixels = dipToPixels(DEFAULT_LR_PADDING_DIP);
		setPadding(paddingPixels, 0, paddingPixels, 0);
		setTextColor(DEFAULT_TEXT_COLOR);
		
		fadeIn = new AlphaAnimation(0, 1);
		fadeIn.setInterpolator(new DecelerateInterpolator());
		fadeIn.setDuration(200);

		fadeOut = new AlphaAnimation(1, 0);
		fadeOut.setInterpolator(new AccelerateInterpolator());
		fadeOut.setDuration(200);
		
		isShown = false;
		
		if (this.target != null) {
			applyTo(this.target);
		} else {
			show();
		}
		
	}

	//指定badge用于的目标view
	private void applyTo(View target) {
		
		LayoutParams lp = target.getLayoutParams();
		ViewParent parent = target.getParent();
		FrameLayout container = new FrameLayout(context);
		
		if (target instanceof TabWidget) {
			
			// set target to the relevant tab child container 为子类容器设置target
			target = ((TabWidget) target).getChildTabViewAt(targetTabIndex);
			this.target = target;
			
			((ViewGroup) target).addView(container, 
					new LayoutParams(LayoutParams.FILL_PARENT, LayoutParams.FILL_PARENT));
			
			this.setVisibility(View.GONE);
			container.addView(this);
			
		} else {
			
			// TODO verify that parent is indeed a ViewGroup
			//确认父类view是可用的
			ViewGroup group = (ViewGroup) parent; 
			int index = group.indexOfChild(target);
			
			group.removeView(target);
			group.addView(container, index, lp);
			
			container.addView(target);
	
			this.setVisibility(View.GONE);
			container.addView(this);
			
			group.invalidate();
			
		}
		
	}
	
	/**
     * Make the badge visible in the UI.
     * 显示badge，最后会用到
     */
	public void show() {
		show(false, null);
	}
	
	/**
     * Make the badge visible in the UI.
     *
     * @param animate flag to apply the default fade-in animation. 是否显示动画效果
	 *
     */
	public void show(boolean animate) {
		show(animate, fadeIn);
	}
	
	/**
     * Make the badge visible in the UI.
     *
     * @param anim Animation to apply to the view when made visible.以anim的动画效果显示
     */
	public void show(Animation anim) {
		show(true, anim);
	}
	
	/**
     * Make the badge non-visible in the UI.
     * 隐藏badge
     */
	public void hide() {
		hide(false, null);
	}
	
	/**
     * Make the badge non-visible in the UI.
     *
     * @param animate flag to apply the default fade-out animation.
     */
	public void hide(boolean animate) {
		hide(animate, fadeOut);
	}
	
	/**
     * Make the badge non-visible in the UI.
     *
     * @param anim Animation to apply to the view when made non-visible.
     */
	public void hide(Animation anim) {
		hide(true, anim);
	}
	
	/**
     * Toggle the badge visibility in the UI.
     * 
     */
	public void toggle() {
		toggle(false, null, null);
	}
	
	/**
     * Toggle the badge visibility in the UI.
     * 
     * @param animate flag to apply the default fade-in/out animation.
     */
	public void toggle(boolean animate) {
		toggle(animate, fadeIn, fadeOut);
	}
	
	/**
     * Toggle the badge visibility in the UI.
     *
     * @param animIn Animation to apply to the view when made visible.
     * @param animOut Animation to apply to the view when made non-visible.
     */
	public void toggle(Animation animIn, Animation animOut) {
		toggle(true, animIn, animOut);
	}
	
	private void show(boolean animate, Animation anim) {
		if (getBackground() == null) {
			if (badgeBg == null) {
				badgeBg = getDefaultBackground();
			}
			setBackgroundDrawable(badgeBg);
		}
		applyLayoutParams();
		
		if (animate) {
			this.startAnimation(anim);
		}
		this.setVisibility(View.VISIBLE);
		isShown = true;
	}
	
	private void hide(boolean animate, Animation anim) {
		this.setVisibility(View.GONE);
		if (animate) {
			this.startAnimation(anim);
		}
		isShown = false;
	}
	
	private void toggle(boolean animate, Animation animIn, Animation animOut) {
		if (isShown) {
			hide(animate && (animOut != null), animOut);	
		} else {
			show(animate && (animIn != null), animIn);
		}
	}
	
	/**
     * Increment the numeric badge label. If the current badge label cannot be converted to
     * an integer value, its label will be set to "0".
     * 增加显示的数字，如果不能显示为数字，则默认显示O
     * @param offset the increment offset.
     */
	public int increment(int offset) {
		CharSequence txt = getText();
		int i;
		if (txt != null) {
			try {
				i = Integer.parseInt(txt.toString());
			} catch (NumberFormatException e) {
				i = 0;
			}
		} else {
			i = 0;
		}
		i = i + offset;
		setText(String.valueOf(i));
		return i;
	}
	
	/**
     * Decrement the numeric badge label. If the current badge label cannot be converted to
     * an integer value, its label will be set to "0".
     * 
     * @param offset the decrement offset.
     */
	public int decrement(int offset) {
		return increment(-offset);
	}
	
	private ShapeDrawable getDefaultBackground() {
		
		int r = dipToPixels(DEFAULT_CORNER_RADIUS_DIP);
		float[] outerR = new float[] {r, r, r, r, r, r, r, r};
        
		RoundRectShape rr = new RoundRectShape(outerR, null, null);
		ShapeDrawable drawable = new ShapeDrawable(rr);
		drawable.getPaint().setColor(badgeColor);
		
		return drawable;
		
	}
	
	private void applyLayoutParams() {
		
		FrameLayout.LayoutParams lp = new FrameLayout.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
		
		switch (badgePosition) {
		case POSITION_TOP_LEFT:
			lp.gravity = Gravity.LEFT | Gravity.TOP;
			lp.setMargins(badgeMarginH, badgeMarginV, 0, 0);
			break;
		case POSITION_TOP_RIGHT:
			lp.gravity = Gravity.RIGHT | Gravity.TOP;
			lp.setMargins(0, badgeMarginV, badgeMarginH, 0);
			break;
		case POSITION_BOTTOM_LEFT:
			lp.gravity = Gravity.LEFT | Gravity.BOTTOM;
			lp.setMargins(badgeMarginH, 0, 0, badgeMarginV);
			break;
		case POSITION_BOTTOM_RIGHT:
			lp.gravity = Gravity.RIGHT | Gravity.BOTTOM;
			lp.setMargins(0, 0, badgeMarginH, badgeMarginV);
			break;
		case POSITION_CENTER:
			lp.gravity = Gravity.CENTER;
			lp.setMargins(0, 0, 0, 0);
			break;
		default:
			break;
		}
		
		setLayoutParams(lp);
		
	}

	/**
     * Returns the target View this badge has been attached to.
     * 
     */
	public View getTarget() {
		return target;
	}

	/**
     * Is this badge currently visible in the UI?
     * badge是否可见
     */
	@Override
	public boolean isShown() {
		return isShown;
	}

	/**
     * Returns the positioning of this badge.
     * 获取badge的位置信息
     * one of POSITION_TOP_LEFT, POSITION_TOP_RIGHT, POSITION_BOTTOM_LEFT, POSITION_BOTTOM_RIGHT, POSTION_CENTER.
     * 
     */
	public int getBadgePosition() {
		return badgePosition;
	}

	/**
     * Set the positioning of this badge.
     * 设置badge的位置
     * @param layoutPosition one of POSITION_TOP_LEFT, POSITION_TOP_RIGHT, POSITION_BOTTOM_LEFT, POSITION_BOTTOM_RIGHT, POSTION_CENTER.
     * 
     */
	public void setBadgePosition(int layoutPosition) {
		this.badgePosition = layoutPosition;
	}

	/**
     * Returns the horizontal margin from the target View that is applied to this badge.
     * 获取badge水平方向上的距离
     */
	public int getHorizontalBadgeMargin() {
		return badgeMarginH;
	}
	
	/**
     * Returns the vertical margin from the target View that is applied to this badge.
     * 获取badge垂直方向上的距离
     */
	public int getVerticalBadgeMargin() {
		return badgeMarginV;
	}

	/**
     * Set the horizontal/vertical margin from the target View that is applied to this badge.
     * 如果水平和垂直方向的距离相同，则可以用该方法设置
     * @param badgeMargin the margin in pixels.
     */
	public void setBadgeMargin(int badgeMargin) {
		this.badgeMarginH = badgeMargin;
		this.badgeMarginV = badgeMargin;
	}
	
	/**
     * Set the horizontal/vertical margin from the target View that is applied to this badge.
     * 设置badge水平和垂直方向的距离
     * @param horizontal margin in pixels. 水平距离 px
     * @param vertical margin in pixels.   垂直距离 px
     */
	public void setBadgeMargin(int horizontal, int vertical) {
		this.badgeMarginH = horizontal;
		this.badgeMarginV = vertical;
	}
	
	/**
     * Returns the color value of the badge background.
     * 返回badge的颜色值
     */
	public int getBadgeBackgroundColor() {
		return badgeColor;
	}

	/**
     * Set the color value of the badge background.
     * 设置badge的颜色值
     * @param badgeColor the badge background color.
     */
	public void setBadgeBackgroundColor(int badgeColor) {
		this.badgeColor = badgeColor;
		badgeBg = getDefaultBackground();
	}
	
	private int dipToPixels(int dip) {
		Resources r = getResources();
		float px = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dip, r.getDisplayMetrics());
		return (int) px;
	}

}

           
            
            
            
