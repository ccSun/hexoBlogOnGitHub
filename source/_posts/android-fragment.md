title: Android-Fragment
categories:
  - Android
tags:
  - Anroid
  - Fragment
date: 2016-03-11 17:08:09
---
收录于Fragment相关的内容，如生命周期，如何添加到activity，fragment通讯等。
## 一、 Fragmentst Lifecycle

1. A fragment must always be embedded in an activity and the fragment's lifecycle is directly affected by the host activity's lifecycle. For example, when the activity is paused, so are all fragments in it, and when the activity is destroyed, so are all fragments. 
2. You can also ***add it to a back stack***. The back stack allows the user to reverse a fragment transaction (navigate backwards), by pressing the Back button.
3. ***onCreate()*** : initialize essential components of the fragment that you want to retain when the fragment is paused or stopped, then resumed.除了view之外的做出实话
4. ***onCreateView()*** : return a View from this method that is the root of your fragment's layout.
5. You can ***save the state during the fragment's onSaveInstanceState()*** callback and ***restore it during either onCreate(), onCreateView(), or onActivityCreated()***.
6. 只有Activity resume之后，才能操作fragment，否则fragment生命周期跟随anctivity不能操作，如在onPause，onStop等。

![Fragmentst Lifecycle](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-fragment/fragment_lifecycle.png?raw=true)   

## 二、 subclasses:
* DialogFragment
* ListFragment
* PreferenceFragment


## 三、Adding a fragment to an activity

### 1. Declare the fragment inside the activity's layout file.

When you add a fragment to an activity layout by defining the fragment in the layout XML file, you cannot remove the fragment at runtime. 

```
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	android:orientation="horizontal"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent">
    	<fragment android:name="com.example.news.ArticleReaderFragment"
            android:id="@+id/viewer"
            android:layout_weight="2"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
		</LinearLayout>
```
		
Each fragment requires a unique identifier that the system can use to restore the fragment if the activity is restarted (and which you can use to capture the fragment to perform transactions, such as remove it). There are ***three ways to provide an ID for a fragment***:

   * Supply the android:id attribute with a unique ID.
   * Supply the android:tag attribute with a unique string.
   * If you provide neither of the previous two, the system uses the ID of the container view.
   


### 2. programmatically add the fragment to an existing ViewGroup.
```
    	FragmentManager fragmentManager = getFragmentManager();
    	FragmentTransaction fragmentTransaction = 		fragmentManager.beginTransaction();
    	ExampleFragment fragment = new ExampleFragment();
    	fragmentTransaction.add(R.id.fragment_container, fragment);
    	fragmentTransaction.commit();
```
### 3、Adding a fragment without a UI

1. you can also use a fragment to ***provide a background behavior for the activity without presenting additional UI***.
2. To add a fragment without a UI, add the fragment from the activity using ***add(Fragment, String) (supplying a unique string "tag" for the fragment, rather than a view ID)***. This adds the fragment, but, because it's not associated with a view in the activity layout, ***it does not receive a call to onCreateView(). So you don't need to implement that method***.
3. If you want to get the fragment from the activity later, you need to use ***findFragmentByTag()***.

## 四、 Performing Fragment Transactions

1.  using methods such as add(), remove(), and replace(). Then, to apply the transaction to the activity, ***you must call commit()***.
2.  you might want to call ***addToBackStack()***, in order to add the transaction to a back stack of fragment transactions. ***This back stack is managed by the activity and allows the user to return to the previous fragment state, by pressing the Back button***. 

    ```
		// Replace whatever is in the fragment_container view with this fragment,
		// and add the transaction to the back stack
		transaction.replace(R.id.fragment_container, newFragment);
		transaction.addToBackStack(null);
    ```
3. If you ***add multiple changes to the transaction*** (such as another add() or remove()) and call addToBackStack(), ***then all changes applied before you call commit() are added to the back stack as a single transaction and the Back button will reverse them all together***.
4. If you ***do not call addToBackStack() when you perform a transaction that removes a fragment, then that fragment is destroyed*** when the transaction is committed and the user cannot navigate back to it.
5. For each fragment transaction, ***you can apply a transition animation, by calling setTransition() before you commit***.
***Caution:*** You can commit a transaction ***using commit() only prior to the activity saving its state*** (when the user leaves the activity). If you attempt to commit after that point, an exception will be thrown. This is because the state after the commit can be lost if the activity needs to be restored. ***For situations in which its okay that you lose the commit, use commitAllowingStateLoss()***.

## 五、 Communicating with Other Fragments

1.  the fragment can access the Activity instance with ***getActivity()***；
2.  your activity can call methods in the fragment by acquiring a reference to the Fragment from FragmentManager, using ***findFragmentById() or findFragmentByTag()***.    
**All Fragment-to-Fragment communication is done through the associated Activity. Two Fragments should never communicate directly.**
2. define an interface in the Fragment class and implement it within the Activity. The Fragment captures the interface implementation during its onAttach() lifecycle method. Then the fragment can deliver messages to the activity by calling the onArticleSelected() method

```
	public class HeadlinesFragment extends ListFragment {
    	OnHeadlineSelectedListener mCallback;

    	// Container Activity must implement this interface
    	public interface OnHeadlineSelectedListener {
        	public void onArticleSelected(int position);
    	}

    	@Override
    	public void onAttach(Activity activity) {
        	super.onAttach(activity);
        
        	// This makes sure that the container activity has implemented
        	// the callback interface. If not, it throws an exception
        	try {
            	mCallback = (OnHeadlineSelectedListener) activity;
        	} catch (ClassCastException e) {
            	throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        	}
    	}
    
    	...
	}
```
```
	public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    	...
    
    	public void onArticleSelected(int position) {
        	// The user selected the headline of an article from the HeadlinesFragment
        	// Do something here to display that article
    	}
	}
```

### 六、Adding items to the App Bar( and Options Menu)

Still not used.
