Dialog的使用

Dialog的使用

***[参考]()***

	package com.qiantu.youqian.module.loan.view;

	*/**
	 * Created by sunxiang on 2018/9/29.
	 */*

	public class BottomAndMaxWidthAndOneButtonDialog extends Dialog {

	    private TextView mTitle, mBody;
	    private ImageView mImageview;
	    private Button mButton;

	    private Callback mCallback;

	    public BottomAndMaxWidthAndOneButtonDialog(@NonNull Context context) {
	        super(context);
	        requestWindowFeature(Window.FEATURE_NO_TITLE);
	    }

	    public BottomAndMaxWidthAndOneButtonDialog(@NonNull Context context, Callback callback) {
	        super(context);
	        requestWindowFeature(Window.FEATURE_NO_TITLE);
	        this.mCallback = callback;

	    }

	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        Log.d("sunxiang", "onCreate: sunxiang dialog");
	        setContentView(R.layout.dialog_title_and_one_button_on_bottom);
	        mTitle = findViewById(R.id.sample_title);
	        mBody = findViewById(R.id.sample_body);
	        mImageview = findViewById(R.id.sample_image);
	        mButton = findViewById(R.id.sample_confirm_button);
	        *//初始化界面控件的事件*
	        initEvent();
	    }

	    private void initEvent() {
	        mButton.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                dismiss();
	                if(mCallback != null) {
	                    mCallback.confirm();
	                }

	            }
	        });
	    }

	    public interface Callback {
	        void confirm();
	    }

	    public void setTitle(String title) {
	        mTitle.setText(title);
	    }

	    public void setBody(String body) {
	        mBody.setText(body);
	    }

	    public void setImageviewResource(int id) {
	        mImageview.setImageResource(id);
	    }

	    public void setImageviewBitmap(Bitmap bitmap) {
	        mImageview.setImageBitmap(bitmap);
	    }

	    public void setImageviewDrawable(Drawable drawable) {
	        mImageview.setImageDrawable(drawable);
	    }

	    public void setButtonText(String text) {
	        mButton.setText(text);
	    }

	    public void setCallback(Callback callback) {
	        this.mCallback = callback;

	    }

	    public ImageView getImageView() {
	        return mImageview;
	    }

	    @Override
	    public void show() {
	        Window dialogWindow = getWindow();
	        dialogWindow.getDecorView().setPadding(0,0,0,0);
	        *//设置Dialog从窗体底部弹出*
	        dialogWindow.setGravity( Gravity.BOTTOM);
	        *//获得窗体的属性*
	        WindowManager.LayoutParams lp = dialogWindow.getAttributes();
	        lp.width = WindowManager.LayoutParams.MATCH_PARENT;
	        lp.height = WindowManager.LayoutParams.WRAP_CONTENT;
	*//        lp.y = 0;//设置Dialog距离底部的距离*
	        *//       将属性设置给窗体*
	        dialogWindow.setAttributes(lp);
	        super.show();

	    }
	}

%0A***%5B%E5%8F%82%E8%80%83%5D(https%3A%2F%2Fwww.jianshu.com%2Fp%2F5b34c1a72d03)***%0A%60%60%60java%0Apackage%20com.qiantu.youqian.module.loan.view%3B%0A%0A%2F**%0A%20*%20Created%20by%20sunxiang%20on%202018%2F9%2F29.%0A%20*%2F%0A%0Apublic%20class%20BottomAndMaxWidthAndOneButtonDialog%20extends%20Dialog%20%7B%0A%0A%20%20%20%20private%20TextView%20mTitle%2C%20mBody%3B%0A%20%20%20%20private%20ImageView%20mImageview%3B%0A%20%20%20%20private%20Button%20mButton%3B%0A%0A%20%20%20%20private%20Callback%20mCallback%3B%0A%0A%20%20%20%20public%20BottomAndMaxWidthAndOneButtonDialog(%40NonNull%20Context%20context)%20%7B%0A%20%20%20%20%20%20%20%20super(context)%3B%0A%20%20%20%20%20%20%20%20requestWindowFeature(Window.FEATURE_NO_TITLE)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20BottomAndMaxWidthAndOneButtonDialog(%40NonNull%20Context%20context%2C%20Callback%20callback)%20%7B%0A%20%20%20%20%20%20%20%20super(context)%3B%0A%20%20%20%20%20%20%20%20requestWindowFeature(Window.FEATURE_NO_TITLE)%3B%0A%20%20%20%20%20%20%20%20this.mCallback%20%3D%20callback%3B%0A%0A%20%20%20%20%7D%0A%0A%0A%20%20%20%20%40Override%0A%20%20%20%20protected%20void%20onCreate(Bundle%20savedInstanceState)%20%7B%0A%20%20%20%20%20%20%20%20super.onCreate(savedInstanceState)%3B%0A%20%20%20%20%20%20%20%20Log.d(%22sunxiang%22%2C%20%22onCreate%3A%20sunxiang%20dialog%22)%3B%0A%20%20%20%20%20%20%20%20setContentView(R.layout.dialog_title_and_one_button_on_bottom)%3B%0A%20%20%20%20%20%20%20%20mTitle%20%3D%20findViewById(R.id.sample_title)%3B%0A%20%20%20%20%20%20%20%20mBody%20%3D%20findViewById(R.id.sample_body)%3B%0A%20%20%20%20%20%20%20%20mImageview%20%3D%20findViewById(R.id.sample_image)%3B%0A%20%20%20%20%20%20%20%20mButton%20%3D%20findViewById(R.id.sample_confirm_button)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E5%88%9D%E5%A7%8B%E5%8C%96%E7%95%8C%E9%9D%A2%E6%8E%A7%E4%BB%B6%E7%9A%84%E4%BA%8B%E4%BB%B6%0A%20%20%20%20%20%20%20%20initEvent()%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20private%20void%20initEvent()%20%7B%0A%20%20%20%20%20%20%20%20mButton.setOnClickListener(new%20View.OnClickListener()%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%40Override%0A%20%20%20%20%20%20%20%20%20%20%20%20public%20void%20onClick(View%20v)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20dismiss()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if(mCallback%20!%3D%20null)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20mCallback.confirm()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20interface%20Callback%20%7B%0A%20%20%20%20%20%20%20%20void%20confirm()%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setTitle(String%20title)%20%7B%0A%20%20%20%20%20%20%20%20mTitle.setText(title)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setBody(String%20body)%20%7B%0A%20%20%20%20%20%20%20%20mBody.setText(body)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setImageviewResource(int%20id)%20%7B%0A%20%20%20%20%20%20%20%20mImageview.setImageResource(id)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setImageviewBitmap(Bitmap%20bitmap)%20%7B%0A%20%20%20%20%20%20%20%20mImageview.setImageBitmap(bitmap)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setImageviewDrawable(Drawable%20drawable)%20%7B%0A%20%20%20%20%20%20%20%20mImageview.setImageDrawable(drawable)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setButtonText(String%20text)%20%7B%0A%20%20%20%20%20%20%20%20mButton.setText(text)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20void%20setCallback(Callback%20callback)%20%7B%0A%20%20%20%20%20%20%20%20this.mCallback%20%3D%20callback%3B%0A%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20ImageView%20getImageView()%20%7B%0A%20%20%20%20%20%20%20%20return%20mImageview%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20void%20show()%20%7B%0A%20%20%20%20%20%20%20%20Window%20dialogWindow%20%3D%20getWindow()%3B%0A%20%20%20%20%20%20%20%20dialogWindow.getDecorView().setPadding(0%2C0%2C0%2C0)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E8%AE%BE%E7%BD%AEDialog%E4%BB%8E%E7%AA%97%E4%BD%93%E5%BA%95%E9%83%A8%E5%BC%B9%E5%87%BA%0A%20%20%20%20%20%20%20%20dialogWindow.setGravity(%20Gravity.BOTTOM)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E8%8E%B7%E5%BE%97%E7%AA%97%E4%BD%93%E7%9A%84%E5%B1%9E%E6%80%A7%0A%20%20%20%20%20%20%20%20WindowManager.LayoutParams%20lp%20%3D%20dialogWindow.getAttributes()%3B%0A%20%20%20%20%20%20%20%20lp.width%20%3D%20WindowManager.LayoutParams.MATCH_PARENT%3B%0A%20%20%20%20%20%20%20%20lp.height%20%3D%20WindowManager.LayoutParams.WRAP_CONTENT%3B%0A%2F%2F%20%20%20%20%20%20%20%20lp.y%20%3D%200%3B%2F%2F%E8%AE%BE%E7%BD%AEDialog%E8%B7%9D%E7%A6%BB%E5%BA%95%E9%83%A8%E7%9A%84%E8%B7%9D%E7%A6%BB%0A%20%20%20%20%20%20%20%20%2F%2F%20%20%20%20%20%20%20%E5%B0%86%E5%B1%9E%E6%80%A7%E8%AE%BE%E7%BD%AE%E7%BB%99%E7%AA%97%E4%BD%93%0A%20%20%20%20%20%20%20%20dialogWindow.setAttributes(lp)%3B%0A%20%20%20%20%20%20%20%20super.show()%3B%0A%0A%20%20%20%20%7D%0A%7D%0A%0A%60%60%60%0A