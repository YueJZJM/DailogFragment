对话框能引起用户的注意，也可以接收用户的输入。我们为应用添加对话框，可以修改日期。如图：

![对话框](https://upload-images.jianshu.io/upload_images/10095597-abf69ea24152acf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/250)

图中的对话框是 AlertDialog 类的一个实例。AlertDialog 类是常用的多用途 Dialog 子类。
学习了 AlertDialog，在此总结一下，同时也包括 fragment 间数据传输的问题。

- 创建 DailogFragment
- 显示 DialogFragment
- 设置对话框显示内容
- fragment 间数据的传递

## 创建 DailogFragment
一般是建议将 AlertDialog 封装在DialogFragment （Fragment 的子类）实例中使用，这样可以使用 FragementManager 管理对话框，可以更加灵活的显示对话框。当然，不使用 DialogFragment 也可以显示 AlertDialog 视图。
另外，如果旋转设备， AlertDialog 会消失，而封装在 fragment 中的 AlertDialog 则不会出现此问题。

要显示对话框，首先完成以下任务：
- 创建名为 DatePickerFragment 的 DialogFragment 子类
- 创建 AlertDialog
- 借助 FragmentManager 在屏幕上显示对话框。

先为对话框添加资源

**value/strings.xml**
```
<string name="date_picker_title">Date of crime:</string>
```
创建 DialogFragment，选择 AppCompat 的版本是 android.support.v7.app.AlertDialog

**DatePickerFragment.java**
```
public class DatePickerFragment extends DialogFragment {
    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        return new AlertDialog.Builder(getActivity())
                .setTitle(R.string.date_picker_title)
                .setPositiveButton(android.R.string.ok,null)
                .create();
    } 
}
```
在以上代码中，以流接口的形式创建了 AlertDialog 实例。首先，将 Context 参数传入 AlertDialog.Builder 类的方法中，返回一个 AlertDialog.Builder 实例。然后调用 AlertDialog.setTitle(...) 和 AlertDialog.setPositiveButton(int, DialogInterface.OnclickLister)，setPositiveButton(...) 传入的两个参数分别是字符串资源和 DialogInterface.OnclickLister 接口的对象。代码中传入的资源 ID 是 Android 的 OK 常量；至于监听器等会儿实现，先传入 null。最后调用 AlertDialog.creat() 返回配置的 AlertDialog 实例。
Android 中有3种用于对话的按钮：positivi、negative、neutral，代表确定，否定，和中立
## 显示 DialogFragment
和其他 fragment 一样，DialogFragment 实例也是由托管 activity 的FragmentManager 管理的，要将 DialogFragment 添加给 FragmentManager 管理并放置到屏幕上，可调用以下两个方法：
```
public void show(FragmentManager, String);
public void show(FragmentTransaction, String);
```
String 是可唯一识别 FragmentManager 队列中的 DialogFragment。两个方法都行，但是 传入 FragmentTransaction，需要自己负责创建事务并提交；传入 FragmentManager 参数，系统会自动创建并提交事务。
这里我们传入的是 FragmentManager。

**CrimeFragment.java**
```
public class CrimeFragment extends Fragment {

private static final String DIALOG_DATE = "DialogDate";

@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
mDateButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                FragmentManager manager = getFragmentManager();
                DatePickerFragment dialog = new DatePickerFragment();
                dialog.show(manager,DIALOG_DATE);
            }
        });
}
```
到这里我们运行程序，已经能看到对话框了。
## 设置对话框显示内容
接下来，使用 AlertDialog.Build 的 setView(...) 方法，给 AlertDialog 对话框添加 DatePicker 组件。
先来设置 DatePicker 的布局，创建 dialog_date.xml 布局文件。

**layout/dialog_date.xml**
```
<DatePicker xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/dialog_date_picker"
    android:calendarViewShown="false">
</DatePicker>
``` 
实例化 DialogPicker 视图并添加到对话框。

**DatePickerFragment .java**
```
public class DatePickerFragment extends DialogFragment {
    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        View view = LayoutInflater.from(getActivity()).inflate(R.layout.dialog_date,null);
        return new AlertDialog.Builder(getActivity())
                .setView(view)
                .setTitle(R.string.date_picker_title)
                .setPositiveButton(android.R.string.ok,null)
                .create()
    }
}
```
现在运行程序，就会出现文章开头的截图了，最后再来看数据的传输问题。
## fragment 间数据的传递
现在需要实现同一个 activity 托管的两个 fragment 之间的数据传递。

![CrimeFragment 与 DatePickerFragment 的对话](https://upload-images.jianshu.io/upload_images/10095597-2c588486625b941a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


首先来看传递数据给 DatePickerFragment。要传递 crime 日期给 DatePickerFragment，需要将它保存在 DatePickerFragment 的 argument bundle 中。创建和设置 fragment argument 通常是在 newInstance() 方法中完成。

**DatePickerFragment.java**
```
public static final String EXTRA_DATE = "con.bingnerdranch.android.criminalintent.date";

private DatePicker mDatePicker;

public static DatePickerFragment newInstance(Date date){
      Bundle args = new Bundle();
      args.putSerializable(ARG_DATE,date);
      DatePickerFragment fragment = new DatePickerFragment();
      fragment.setArguments(args);
     return  fragment;
}
``` 
然后，在 CrimeFragment 中，用 DatePickerFragment.newInstance(Date) 方法替换 DatePickerFragment 的构造方法。

**CrimeFragment.java**
```
public class CrimeFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
    mDateButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                FragmentManager manager = getFragmentManager();
               // DatePickerFragment dialog = new DatePickerFragment();
                DatePickerFragment dialog = DatePickerFragment.newInstance(mCrime.getDate());
                dialog.setTargetFragment(CrimeFragment.this,REQUEST_DATE);
                dialog.show(manager,DIALOG_DATE);
    }
}
```
DatePickerDialogFragment 使用的 Date 中的信息来初始化 DatePicker 对象。但是 DatePicker 对象的初始化需要整数形式的年月日。Date 是时间戳，无法直接提供整数，我们还需要创建一个 Calender 对象，用 Date 对象配置它，获得年月日。

**DatePickerFragment.java**
```
public class DatePickerFragment extends DialogFragment {
    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        final Date date = (Date) getArguments().getSerializable(ARG_DATE);

        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        int year = calendar.get(Calendar.YEAR);
        int month = calendar.get(Calendar.MONTH);
        int day = calendar.get(Calendar.DAY_OF_MONTH);
        View view = LayoutInflater.from(getActivity()).inflate(R.layout.dialog_date,null);

        mDatePicker = (DatePicker) view.findViewById(R.id.dialog_date_picker);
        mDatePicker.init(year,month,day,null);
        ...
    ｝
}
```
现在可以从 CrimeFragment 向 DatePickerFragment 传递信息了。还需要返回在 DatePickerFragment 中设置的数据。

如果是 activity 的数据回传，我们可以调用 startActivityForResult(...) 方法，ActivityManager 负责跟踪管理 activity 父子关系。回传数据后，子 activity 被销毁，但 ActivityManager 知道接收数据的是哪个 activity。

类似于 activity 间的关联，可将 C日么Fragment 设置成 DatePickerFragment 的目标 fragment。这样在 CrimeFragment 和 DatePickerFragment 被销毁并重建后，操作系统会重新关联它们。调用一下方法就能建立关联：
```
public void setTargetFragment(Fragment fragment, int requestCode)
```
先来设置目标fragment，将 CrimeFragment 设为 DatePickerFragment 实例的目标 fragment。

**CrimeFragment.java**
```
public class CrimeFragment extends Fragment {

    private static final String ARG_CRIME_ID = "crime_id";
    private static final String DIALOG_DATE = "DialogDate";

    private static final int REQUEST_DATE = 0;
    ...
     @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        mDateButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                FragmentManager manager = getFragmentManager();
                DatePickerFragment dialog = DatePickerFragment.newInstance(mCrime.getDate());
                dialog.setTargetFragment(CrimeFragment.this,REQUEST_DATE);
                dialog.show(manager,DIALOG_DATE);
            }
        });
    }
}
```
现在要把数据回传给 CrimeFragment。回传日期将作为 extra 附加给 Intent。我们怎么回传呢？让 DatePickerFragment 类调用 CrimeFragment.onActivityResult(int, int, Intent)，此方法是 ActivityManager 在子 activity 被销毁后调用的父 activity 方法。处理 activity 间的数据返回时，ActivityManager 会自动调用 Activity.onActivityResult(...) 方法。父 activity 接收到 Activity.onActivityResult(...) 方法调用后，其 FragmentManager 会调用对用的 fragment 的Fragment.onActivityResult(...) 方法。

处理由同一 activity 托管的两个 fragment 间的数据返回时，可借用 Fragment.onActivityResult(...) 方法。

下面是实现步骤：
- 在 DatePickerFragment 类中，新建 sendResult(...) 私有方法，创建 intent 并将日期数据作为 extra 附加到 intent 上。最后调用 CrimeFragment.onActivityResult(...) 方法。

**DatePickerFragment.java**
```
public class DatePickerFragment extends DialogFragment {

    public static final String EXTRA_DATE = "con.bingnerdranch.android.criminalintent.date";
    
    private void sendResult(int resultCode,Date date){
        if (getTargetFragment() == null){
            return;
        }
        Intent intent = new Intent();
        intent.putExtra(EXTRA_DATE,date);
        getTargetFragment().onActivityResult(getTargetRequestCode(),resultCode,intent);
    }
}
```
- 现在来使用 sendResult(...)，在用户点击对话框中的 positive 按钮时，需要从 DatePicker 中获取日期并回传给 CrimeFragment。

**DatePickerFragment**
```
public class DatePickerFragment extends DialogFragment {
    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
       
        return new AlertDialog.Builder(getActivity())
                .setView(view)
                .setTitle(R.string.date_picker_title)
               // .setPositiveButton(android.R.string.ok,null)
                .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        int year = mDatePicker.getYear();
                        int month = mDatePicker.getMonth();
                        int day = mDatePicker.getDayOfMonth();
                        Date date = new GregorianCalendar(year,month,day).getTime();
                        sendResult(Activity.RESULT_OK,date1);
                    }
                })
                .create();
    }
}
```
- 在 CrimeFragment 中，覆盖 onActivityResult(...) 方法，从 extra 中获取日期数据。

**CrimeFragment.java**
```
public class CrimeFragment extends Fragment {
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode != Activity.RESULT_OK){
            return;
        }

        if (requestCode == REQUEST_DATE){
            Date date = (Date) data.getSerializableExtra(DatePickerFragment.EXTRA_DATE);
            mCrime.setDate(date);
            upDate();
        }
    }

    private void upDate() {
        mDateButton.setText(mCrime.getDate().toString());
    }
}
```
现在所有的任务完成了


[简书地址](https://www.jianshu.com/p/4686ae4a3900)
