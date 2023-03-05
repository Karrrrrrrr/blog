---
title: "WPF"
date: 2023-03-06T22:26:42+08:00
---

#  WPF
2023/03/06写一天wpf的一点心得

从设计上和vue还是很像的

和vue的一些对比, vue的响应式是通过proxy实现的, 但是C#没有proxy, 所以绑定model需要自己设置set/get, 从而发射事件,通知主线程重新渲染
## ViewModelBase
```C#
using System.ComponentModel;
using System.Diagnostics;

namespace Manager_v1.pages.models {
    public abstract class ViewModelBase : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;
        protected void RaisePropertyChangedEvent([System.Runtime.CompilerServices.CallerMemberName] string propertyName = "")
        {
            if (propertyName == "")
            {
                propertyName = GetCallerMemberName();
            }
            if (PropertyChanged != null)
            {
                PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
            }
        }
        string GetCallerMemberName()
        {
            StackTrace trace = new StackTrace();
            StackFrame frame = trace.GetFrame(2);//1代表上级，2代表上上级，以此类推  
            var propertyName = frame.GetMethod().Name;
            if (propertyName.StartsWith("get_") || propertyName.StartsWith("set_") || propertyName.StartsWith("put_"))
            {
                propertyName = propertyName.Substring("get_".Length);
            }
            return propertyName;
        }
    }
}
```
```C#
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Data;
using System.Windows;

namespace Manager_v1.pages.models {
    public class DashboardStore : ViewModelBase {
        private string _age;

        public string Age {
            set {
                _age = value;
                RaisePropertyChangedEvent();
            }
            get => _age;
        }

        public ObservableCollection<User> List { get; set; } = new ObservableCollection<User>();

        public DashboardStore() {
           
        }
    }

    public class User {
        public int Age { get; set; } = 123;
        public string Name { get; set; } = "Name 13";
        public string ClassName { get; set; } = "ClassName 13";
    }
}
```
代码参考如上, 所有的响应式实体类都得通过继承ViewModelBase, 在set里面调用RaisePropertyChangedEvent

## v-for 的对比
C#中没有v-for那么灵活的东西, 如果需要实现类似的功能, 首先需要一个支持变化的容器(List, ObservableCollection);

## ListBox
自定义循环组件(style是为了删除点击特效)
```xml
 <ListBox ItemsSource="{Binding List}">
    <ListBox.ItemContainerStyle>
        <Style TargetType="ListBoxItem">
            <!-- <Setter Property="IsSelected" Value="{Binding Content.IsSelected, Mode=TwoWay, RelativeSource={RelativeSource Self}}"/> -->
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ListBoxItem">
                        <ContentPresenter />
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </ListBox.ItemContainerStyle>
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal">
                <TextBlock Margin="10 10 " Text="{Binding Age}" Width="100" />
                <TextBlock Margin="10 10 " Text="{Binding Name}" Width="100" />
                <Button    Margin="10 10 " Content="Delete" Background="Red" Width="100" />
                <!-- <Button Content="{Binding Name}" Width="100" /> -->
                <!-- <Button Content='{Binding ClassName}' Width="100" /> -->
            </StackPanel>
        </DataTemplate>
        <!-- <DataTemplate> -->
        <!--     <StackPanel Orientation="Horizontal"> -->
        <!--         <Button Margin="10 0" Command="{Binding Click}" Content="{Binding Name}" /> -->
        <!--         <Button Margin="10 0" Command="{Binding Click}" Content="{Binding Age}" /> -->
        <!--         <Button Margin="10 0" Command="{Binding Click}" Content="{Binding ClassName}" /> -->
        <!--     </StackPanel> -->
        <!-- </DataTemplate> -->
    </ListBox.ItemTemplate>
</ListBox>
```
## DataGrid
自定义表格: 
```xml
    <DataGrid ItemsSource="{Binding List}" IsReadOnly="True" AutoGenerateColumns="False" CanUserAddRows="False">
    <DataGrid.Columns>

        <DataGridTemplateColumn Header="operate" Width="*">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" Margin="20 10 "
                                   Text="{Binding Name}">
                        </TextBlock>
                    </StackPanel>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <DataGridTemplateColumn Header="operate" Width="*">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" Margin="20 10 "
                                   Text="{Binding Age}">
                        </TextBlock>
                    </StackPanel>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <DataGridTemplateColumn Header="operate" Width="*">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <Button HorizontalAlignment="Center" VerticalAlignment="Center" Margin="20 10 "
                                Click="SayHello" Tag="{Binding  }">
                            CLICK ME
                        </Button>
                    </StackPanel>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

    </DataGrid.Columns>
    <!-- <DataGrid.ItemTemplate> -->
    <!-- <DataTemplate> -->
    <!-- <StackPanel Orientation="Horizontal"> -->
    <!-- <TextBlock Margin="10 10 " Text="{Binding Name}"></TextBlock> -->
    <!-- <TextBlock Margin="10 10 " Text="{Binding Age}"></TextBlock> -->
    <!-- <Button>DELETE ME </Button> -->
    <!-- </StackPanel> -->
    <!-- </DataTemplate> -->
    <!-- </DataGrid.ItemTemplate> -->
</DataGrid>
```

```C#
3. ```


## StackPanel
这是一个类似div的东西

## 关于循环点击事件的绑定
```C#
private void SayHello(object sender, RoutedEventArgs e) { 
    Button btn = (Button)sender;
    User user = (User)btn.Tag; // tag就是参数
}
```
```xml
<StackPanel Orientation="Horizontal">
    <Button HorizontalAlignment="Center" VerticalAlignment="Center" Margin="20 10 " Click="SayHello" Tag="{Binding  }">
        CLICK ME
    </Button>
</StackPanel>

```


# Material 初始化

App.xaml 
```xml
<Application x:Class="Manager_v1.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:Manager_v1"
             xmlns:materialDesign="http://materialdesigninxaml.net/winfx/xaml/themes"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <materialDesign:BundledTheme BaseTheme="Light" PrimaryColor="DeepPurple" SecondaryColor="Lime" />
                <ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.Wpf;component/Themes/MaterialDesignTheme.Defaults.xaml" />
                <ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.Wpf;component/Themes/MaterialDesignTheme.Light.xaml" /> 
                <ResourceDictionary Source="pack://application:,,,/MaterialDesignColors;component/Themes/Recommended/Primary/MaterialDesignColor.DeepPurple.xaml" />
                <ResourceDictionary Source="pack://application:,,,/MaterialDesignColors;component/Themes/Recommended/Accent/MaterialDesignColor.Lime.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>

```


## Frame 
类似router-view, 但是他有个前后箭头, 需要手动去掉
```xml
<Frame Source="./pages/Login.xaml" NavigationUIVisibility="Hidden"></Frame>
```

## 窗口切换
代码如下, new新窗口对象, 关闭自己即可
```C#
private void OnExit(object sender, RoutedEventArgs e) {
    Window dashboard = new MainWindow();
    dashboard.Show();
    Close();
}   
```

## 路由式切换
```C#

private void OnClick(object sender, RoutedEventArgs e) {
    Uri uri = new Uri("./pages/Login.xaml", UriKind.Relative);
    if (NavigationService != null) 
        NavigationService.Source = (uri);
}
```
