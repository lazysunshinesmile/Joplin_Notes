Dialog 设置宽度和高度

Dialog 设置宽度和高度

```java
AlertDialog dialog = builder.create();
dialog.setView(view);
dialog.show();
WindowManager m = getWindowManager();  
Display d = m.getDefaultDisplay();  //为获取屏幕宽、高  
android.view.WindowManager.LayoutParams p = dialog.getWindow().getAttributes();  //获取对话框当前的参数值  
p.height = (int) (d.getHeight() * 0.3);   //高度设置为屏幕的0.3
p.width = (int) (d.getWidth() * 0.5);    //宽度设置为屏幕的0.5 
dialog.getWindow().setAttributes(p);     //设置生效  
```