---
layout: post
title: Optimized TabbedPage with Xamarin.Forms, ReactiveUI and Sextant
date: '2019-06-21 17:00:00'
---

## App Start

Let's start saying that building a tabbed page with Xamarin was always a bit tricky, especially until you understand the concept of navigation stacks.

Assuming that you know how Xamarin.Forms navigation stack works, let's just tackle here the problems related to the tabbed page:

### To create a TabbedPage on Xamarin.Forms, you will need:

1) TabBarNavigationView : ReactiveTabbedPage<TabBarNavigationViewModel>
   
2) Some views:
   * HomeView
   * RedView
   * BlueAView
  
3) The view models for those views:
   * HomeViewModel
   * RedViewModel
   * BlueAViewModel
  
4) A navigation page for each tab, so you are allowed to navigate inside the tabs:
   * OneNavigationView
   * TwoNavigationView
   * ThreeNavigationView
   
5) And since we are talking about Sextant here, you need to provide an instance of viewStackService to each tab.

Considering this as the main page, you need to instantiate TabBarNavigationView and add navigation to each tab:

   1) OneNavigationView => Push an instance of OneAView and OneAViewModel
   
   2) TwoNavigationView => Push an instance of TwoAView and TwoAViewModel
   
   3) ThreeNavigationView => Push an instance of ThreeAView and ThreeAViewModel



Then set this TabBarNavigationView as your MainPage and you will be ready to roll.

Here is how they look as stacks:

![Dfault TabbedPage](/assets/2019-06-21-AppStart.jpg)

Now take some time and notice how all the pages are already created and just waiting to appear.
From a ViewModel-First perspective, it means that the ViewModels for those pages are also already created and anything that runs on the constructor will be already done (for example, loading some items for a list to be ready to display on appearing), which can add a ton of a load time as this TabbedPage is the first page of your app.

This happens because of Xamarin.Forms TabbedPage doesn't support virtualization.

## Optimizing

The solution to this problem is to delay the creating of the View and ViewModels until they Appear, this way only the first View/ViewModel will be created and rendered.

And a way to do that on Xamarin.Forms is creating a very light intermediate View/ViewModels that pushed the real page on appearing. This intermediate View/ViewModel is called TabView/TabViewModel.

So your stack will look like this when starting the app:

![Dfault TabbedPage](/assets/2019-06-21-OptimizedApp.jpg)

Only one real view/ViewModel will get created and rendered, the others are just empty pages that will push the real ones when activated. 

Here in this link where you can find a working sample of this concept:
https://github.com/giusepe/TabbedPageSextantSample