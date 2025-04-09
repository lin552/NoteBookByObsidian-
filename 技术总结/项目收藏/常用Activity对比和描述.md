---
创建时间: 2025-03-14T18:39:00
作者: wangxiaoming
tags:
  - Activity
  - Android
---
##### 各Activity关系图


##### 详细Activity介绍和用途
######  androidx.core.app.ComponentActivity


###### androidx.activity.ComponentActivity


###### androidx.fragment.app.FragmentActivity




###### androidx.appcompat.app.AppCompatActivity



package androidx.activity;
> **ComponentActivity ： androidx.core.app.ComponentActivity**
> 
> Base class for activities that enables composition of higher level components.
> Rather than all functionality being built directly into this class, only the minimal set of lower level building blocks are included. Higher level components can then be used as needed without enforcing a deep Activity class hierarchy or strong coupling between components.


> package androidx.appcompat.app;
> **AppCompatActivity ：FragmentActivity**
> 
> Base class for activities that wish to use some of the newer platform features on older Android devices. Some of these backported features include:
> Using the action bar, including action items, navigation modes and more with the setSupportActionBar(Toolbar) API.
> Built-in switching between light and dark themes by using the Theme. AppCompat. DayNight theme and AppCompatDelegate. setDefaultNightMode(int) API.
> Integration with DrawerLayout by using the getDrawerToggleDelegate() API.
> Note that every activity that extends this class has to be themed with Theme. AppCompat or a theme that extends that theme.

> package androidx.fragment.app;
> **FragmentActivity : ComponentActivity**
> 
> Base class for activities that want to use the support-based Fragments.
> Known limitations:
> When using the <fragment> tag, this implementation can not use the parent view's ID as the new fragment's ID. You must explicitly specify an ID (or tag) in the <fragment>.