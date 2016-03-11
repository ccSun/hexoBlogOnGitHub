title: Android-Fragment
categories:
  - Android
tags:
  - Anroid
  - Fragment
date: 2016-03-11 17:08:09
---

## 一、 Fragmentst Lifecycle

1. A fragment must always be embedded in an activity and the fragment's lifecycle is directly affected by the host activity's lifecycle. For example, when the activity is paused, so are all fragments in it, and when the activity is destroyed, so are all fragments. 
2. You can also add it to a back stack. The back stack allows the user to reverse a fragment transaction (navigate backwards), by pressing the Back button.

### Create a Fragment

1. When you add a fragment to an activity layout by defining the fragment in the layout XML file, you cannot remove the fragment at runtime. 

## 二、 Communicating with Other Fragments

1. All Fragment-to-Fragment communication is done through the associated Activity. Two Fragments should never communicate directly.
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
