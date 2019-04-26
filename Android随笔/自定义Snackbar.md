1. 改变Snackbar的背景  
```java
Snackbar snackbar = Snackbar.make(getBinding().getRoot(), "test", Snackbar.LENGTH_LONG)
                        .setAction("确定", (view) -> {
                            //todo 
                        });
                //取出snackbar view
                View v = snackbar.getView();
                //更换viewweni背景
                v.setBackgroundColor(ContextCompat.getColor(MainActivity.this, R.color.btnBgEnable));
                snackbar.show();
```
2. 修改内容部分的文字颜色
```java
Snackbar snackbar = Snackbar.make(getBinding().getRoot(), "test", Snackbar.LENGTH_LONG)
                        .setAction("确定", (view) -> {
                            //todo 
                        });
                //取出snackbar view
                View v = snackbar.getView();
               // 取出 message textview
              TextView textView = v.findViewById(android.support.design.R.id.snackbar_text);
              // 改变文字顏色
              textView.setTextColor(ContextCompat.getColor(MainActivity.this, R.color.colorSnackbarText));
                snackbar.show();
```
3. 修改Action文字顏色
```java
Snackbar snackbar = Snackbar.make(getBinding().getRoot(), "test", Snackbar.LENGTH_LONG)
                        .setAction("确定", (view) -> {
                            //todo 
                        });
                //取出snackbar view
                View v = snackbar.getView();
               // 取出 action textview
              TextView textView = v.findViewById(android.support.design.R.id.snackbar_action);
              // 改变文字顏色
              textView.setTextColor(ContextCompat.getColor(MainActivity.this, R.color.colorSnackbarText));
                snackbar.show();
```
