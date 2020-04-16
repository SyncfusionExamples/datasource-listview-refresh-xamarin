# How to improve performance when doing bulk changes in Xamarin.Forms ListView (SfListView)

You can improve the performance lag when you make bulk changes with bound collection by suspending and resuming the refresh to ListView with the help of DataSource [BeginInit](https://help.syncfusion.com/cr/xamarin/Syncfusion.DataSource.Portable~Syncfusion.DataSource.DataSource~BeginInit.html?) and [EndInit](https://help.syncfusion.com/cr/xamarin/Syncfusion.DataSource.Portable~Syncfusion.DataSource.DataSource~EndEdit.html?) method in Xamarin.Forms [SfListView](https://help.syncfusion.com/xamarin/listview/overview?).

You can also refer the following article.

https://www.syncfusion.com/kb/11395/how-to-improve-performance-when-doing-bulk-changes-in-xamarin-forms-listview-sflistview

**Step 1:** Install [Refractored.MVVMHelpers](https://www.nuget.org/packages/Refractored.MvvmHelpers) nuget package to use the [ObservableRangeCollection](https://github.com/jamesmontemagno/mvvm-helpers#observablerangecollection) in the shared code project.

**Step 2:** Create the **ObservableRangeCollection** in the ViewModel class.

``` c#
namespace ListViewXamarin
{
    public class ContactsViewModel : BaseViewModel
    {
        public ObservableRangeCollection<Contacts> ContactsInfo { get; set; }
 
        public ContactsViewModel()
        {
            ContactsInfo = new ObservableRangeCollection<Contacts>();
        }
    }
}
```

**Step 3:** Bind the ObservableRangeCollection to the [SfListView.ItemsSource](https://help.syncfusion.com/cr/cref_files/xamarin/Syncfusion.SfListView.XForms~Syncfusion.ListView.XForms.SfListView~ItemsSource.html?) property.

``` xml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:ListViewXamarin"
             xmlns:syncfusion="clr-namespace:Syncfusion.ListView.XForms;assembly=Syncfusion.SfListView.XForms"
             x:Class="ListViewXamarin.MainPage">
    
    <ContentPage.Behaviors>
        <local:Behavior/>
    </ContentPage.Behaviors>
    
  <ContentPage.Content>
        <StackLayout>
            <Button x:Name="addButton" Text="Populate ListView items"/>
            <syncfusion:SfListView x:Name="listView" ItemSpacing="1" ItemSize="60" ItemsSource="{Binding ContactsInfo}">
                <syncfusion:SfListView.ItemTemplate >
                    <DataTemplate>
                        <Grid x:Name="grid">
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="70" />
                                <ColumnDefinition Width="*" />
                            </Grid.ColumnDefinitions>
                            <Image Source="{Binding ContactImage}" VerticalOptions="Center" HorizontalOptions="Center" HeightRequest="50" WidthRequest="50"/>
                            <Grid Grid.Column="1" RowSpacing="1" Padding="10,0,0,0" VerticalOptions="Center">
                                <Label LineBreakMode="NoWrap" TextColor="#474747" Text="{Binding ContactName}"/>
                                <Label Grid.Row="1" Grid.Column="0" TextColor="#474747" LineBreakMode="NoWrap" Text="{Binding ContactNumber}"/>
                            </Grid>
                        </Grid>
                    </DataTemplate>
                </syncfusion:SfListView.ItemTemplate>
            </syncfusion:SfListView>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```
**Step 4:** Added items to the ViewModel collection using the AddRange method. Use the DataSource BeginInit and EndInit method to refresh all the items at once in the UI.

``` c#
namespace ListViewXamarin
{
    public class Behavior : Behavior<ContentPage>
    {
        SfListView ListView;
        Button AddButton;
        ContactsViewModel ViewModel;
 
        protected override void OnAttachedTo(ContentPage bindable)
        {
            ListView = bindable.FindByName<SfListView>("listView");
            AddButton = bindable.FindByName<Button>("addButton");
            AddButton.Clicked += AddButton_Clicked;
 
            base.OnAttachedTo(bindable);
        }
 
        private void AddButton_Clicked(object sender, EventArgs e)
        {
            this.GenerateInfo();
        }
 
        private void GenerateInfo()
        {
            Random r = new Random();
            var contactsInfo = new ObservableCollection<Contacts>();
            for (int i = 0; i < 15; i++)
            {
                var contact = new Contacts(ViewModel.CustomerNames[i], r.Next(720, 799).ToString() + " - " + r.Next(3010, 3999).ToString());
                contact.ContactImage = ImageSource.FromResource("ListViewXamarin.Images.Image" + r.Next(0, 28) + ".png");
                contactsInfo.Add(contact);
            }
 
            ListView.DataSource.BeginInit();
            ViewModel.ContactsInfo.AddRange(contactsInfo);
            ListView.DataSource.EndInit();
        }
    }
}
```
