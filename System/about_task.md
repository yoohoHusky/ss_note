### 判断最后Activity    
- 是否是栈顶activity   
```java
activity.isTaskRoot();
```
- 是否在当前最顶端的activity是否是预期package里    
  ```java
  ActivityManager am = activity.getSystemService(Context.ACTIVITY_SERVICE);
  ComponentName topActivity = am.getRunningTasks(1).get(0).topActivity;
  String currentPkg = topActivity.getPackageName();
  String aimPkg = activity.getPackageName();
  Boolen isInpkg = currentPkg.equlas(aimPkg) ? true : false;
  ```
