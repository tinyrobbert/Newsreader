
实施自适应用户界面流程
根据您的应用当前显示的布局，用户界面流程可能会有所不同。例如，如果您的应用处于双面板模式下，点击左侧面板上的项即可直接在右侧面板上显示相关内容；如果该应用处于单面板模式下，相关内容就应以其他活动的形式在同一面板上显示。

确定当前布局
由于每种布局的实施都会稍有不同，因此您可能需要先确定当前向用户显示的布局。例如，您可以了解用户所处的是“单面板”模式还是“双面板”模式。要做到这一点，您可以查询指定视图是否存在以及是否已显示出来。

public class NewsReaderActivity extends FragmentActivity {
    boolean mIsDualPane;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main_layout);

        View articleView = findViewById(R.id.article);
        mIsDualPane = articleView != null && 
                        articleView.getVisibility() == View.VISIBLE;
    }
}
请注意，这段代码用于查询“报道”面板是否可用，与针对具体布局的硬编码查询相比，这段代码的灵活性要大得多。

再举一个适应各种组件的存在情况的方法示例：在对这些组件执行操作前先查看它们是否可用。例如，新闻阅读器示例应用中有一个用于打开菜单的按钮，但只有在版本低于 3.0 的 Android 上运行该应用时，这个按钮才会存在，因为 API 级别 11 或更高级别中的 ActionBar 已取代了该按钮的功能。因此，您可以使用以下代码为此按钮添加事件侦听器：

Button catButton = (Button) findViewById(R.id.categorybutton);
OnClickListener listener = /* create your listener here */;
if (catButton != null) {
    catButton.setOnClickListener(listener);
}
根据当前布局做出响应
有些操作可能会因当前的具体布局而产生不同的结果。例如，在新闻阅读器示例中，如果用户界面处于双面板模式下，那么点击标题列表中的标题就会在右侧面板中打开相应报道；但如果用户界面处于单面板模式下，那么上述操作就会启动一个独立活动：

@Override
public void onHeadlineSelected(int index) {
    mArtIndex = index;
    if (mIsDualPane) {
        /* display article on the right pane */
        mArticleFragment.displayArticle(mCurrentCat.getArticle(index));
    } else {
        /* start a separate activity */
        Intent intent = new Intent(this, ArticleActivity.class);
        intent.putExtra("catIndex", mCatIndex);
        intent.putExtra("artIndex", index);
        startActivity(intent);
    }
}
同样，如果该应用处于双面板模式下，就应设置带导航标签的操作栏；但如果该应用处于单面板模式下，就应使用旋转窗口小部件设置导航栏。因此您的代码还应确定哪种情况比较合适：

final String CATEGORIES[] = { "热门报道", "政治", "经济", "Technology" };

public void onCreate(Bundle savedInstanceState) {
    ....
    if (mIsDualPane) {
        /* use tabs for navigation */
        actionBar.setNavigationMode(android.app.ActionBar.NAVIGATION_MODE_TABS);
        int i;
        for (i = 0; i < CATEGORIES.length; i++) {
            actionBar.addTab(actionBar.newTab().setText(
                CATEGORIES[i]).setTabListener(handler));
        }
        actionBar.setSelectedNavigationItem(selTab);
    }
    else {
        /* use list navigation (spinner) */
        actionBar.setNavigationMode(android.app.ActionBar.NAVIGATION_MODE_LIST);
        SpinnerAdapter adap = new ArrayAdapter(this, 
                R.layout.headline_item, CATEGORIES);
        actionBar.setListNavigationCallbacks(adap, handler);
    }
}
重复使用其他活动中的片段
多屏幕设计中的重复模式是指，对于某些屏幕配置，已实施界面的一部分会用作面板；但对于其他配置，这部分就会以独立活动的形式存在。例如，在新闻阅读器示例中，对于较大的屏幕，新闻报道文本会显示在右侧面板中；但对于较小的屏幕，这些文本就会以独立活动的形式存在。

在类似情况下，您通常可以在多个活动中重复使用相同的 Fragment 子类以避免代码重复。例如，您在双面板布局中使用了 ArticleFragment：

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
然后又在小屏幕的活动布局中重复使用（无布局）了它 (ArticleActivity)：

ArticleFragment frag = new ArticleFragment();
getSupportFragmentManager().beginTransaction().add(android.R.id.content, frag).commit();
当然，这与在 XML 布局中声明片段的效果是一样的，但在这种情况下却没必要使用 XML 布局，因为报道片段是此活动中的唯一组件。

请务必在设计片段时注意，不要针对具体活动创建强耦合。要做到这一点，您通常可以定义一个界面，该界面概括了相关片段与其主活动交互所需的全部方式，然后让主活动实施该界面：

例如，新闻阅读器应用的 HeadlinesFragment 会精确执行以下代码：

public class HeadlinesFragment extends ListFragment {
    ...
    OnHeadlineSelectedListener mHeadlineSelectedListener = null;

    /* Must be implemented by host activity */
    public interface OnHeadlineSelectedListener {
        public void onHeadlineSelected(int index);
    }
    ...

    public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener listener) {
        mHeadlineSelectedListener = listener;
    }
}
然后，如果用户选择某个标题，相关片段就会通知由主活动指定的侦听器（而不是通知某个硬编码的具体活动）：

public class HeadlinesFragment extends ListFragment {
    ...
    @Override
    public void onItemClick(AdapterView<?> parent, 
                            View view, int position, long id) {
        if (null != mHeadlineSelectedListener) {
            mHeadlineSelectedListener.onHeadlineSelected(position);
        }
    }
    ...
}
支持平板电脑和手持设备的指南中进一步介绍了此技术。

处理屏幕配置变化
如果您使用独立活动实施界面的独立部分，那么请注意，您可能需要对特定配置变化（例如屏幕方向的变化）做出响应，以便保持界面的一致性。

例如，在运行 Android 3.0 或更高版本的标准 7 英寸平板电脑上，如果新闻阅读器示例应用运行在纵向模式下，就会在使用独立活动显示新闻报道；但如果该应用运行在横向模式下，就会使用双面板布局。

也就是说，如果用户处于纵向模式下且屏幕上显示的是用于阅读报道的活动，那么您就需要在检测到屏幕方向变化（变成横向模式）后执行相应操作，即停止上述活动并返回主活动，以便在双面板布局中显示相关内容：

public class ArticleActivity extends FragmentActivity {
    int mCatIndex, mArtIndex;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mCatIndex = getIntent().getExtras().getInt("catIndex", 0);
        mArtIndex = getIntent().getExtras().getInt("artIndex", 0);

        // If should be in two-pane mode, finish to return to main activity
        if (getResources().getBoolean(R.bool.has_two_panes)) {
            finish();
            return;
        }
        ...
}
