# Paging-Knockout
Reusable Paging component using Knockout.js.

**NPM Package** <br/>
    
    npm install paging-knockout

**Features**

 1. **Show on need** <br/> When there is no need for paging at all (for example the items which need to display less than the page size) then the `HTML` component will disappear. <br/>
This will be established by  statement `data-bind="visible: NeedPaging"`.

 2. **Disable on need** <br/>
for example, if you are already selected the last page, why the `last page` or the `Next` button should be available to press? <br/>
*I am handling this and in that case I am disabling those buttons by applying the following binding* `data-bind="css: { disabled: !PreviousPageActive() }"`<br/>

 3. **Distinguish the Selected page** <br/>
a special class (in this case called `active` class) is applied on the selected page, to make the user know in which page he/she is right now. <br/> This is established by the binding `data-bind="css: { active: $parent.CurrentPage() === $data }"`

 4. **Last & First** <br />
going to the first and last page is also available by simple buttons dedicated to this.

 5. **Limits for displayed buttons** <br/>
suppose you have a lot of pages, for example 1000 pages, then what will happened? would you display them all for the user ? *absolutely not* you have to display just a few of them according to the current page. for example showing 3 pages before the page page and other 3 pages after the selected page. <br/>
This case has been handled here `<!-- ko foreach: GetPages() -->` <br>
the `GetPages` function applying a simple algorithm to determine if we need to show all the pages (the page count is under the threshold, which could be determined easily), or to show just some of the buttons. <br/>
you can determine the threshold by changing the value of the `maxPageCount ` variable <br/>
Right now I assigned it as the following `var maxPageCount = 7;` which mean that no more than 7 buttons could be displayed for the user (3 before the SelectedPage, and 3 after the Selected Page) and the Selected Page itself. <br/><br/>
You may wonder, what if there was not enough pages after **OR** before the current page to display? *do not worry I am handling this in the algorithm* <br/> for example, if you have `11 pages` and you have `maxPageCount = 7` and the current `selected page is 10`, Then the following pages will be shown <br/><br/>
`5,6,7,8,9,10(selected page),11` <br/><br/>
so we always stratifying the `maxPageCount`, in the previous example showing `5` pages before the selected page and just `1` page after the selected page.

 6. **Selected Page Validation** <br/>
All set operation for the `CurrentPage` observable *which determine the selected page by the user*, is go through the function `SetCurrentPage`. In only this function we set this observable, and as you can see from the code, before setting the value we make validation operations to make sure that we will not go beyond the available page of the pages.

 7. **Already clean** <br/>
I use only `pureComputed` not `computed` properties, which means you do not need to bother yourself with cleaning and disposing those properties. *Although ,as you will see in example below, you need to dispose some other subscriptions which are outside of the component itself*

**NOTE 1** <br/>
You may noticed that I am using some `bootstrap` classes in this component, 
This is suitable for me, but *of course* you can use your own classes instead of the bootstrap classes. <br/>
The bootstrap classes which I used here are `pagination`, `pagination-sm`, `active` and `disabled` <br/>
Feel free to change them as you need.

**NOTE 2** <br/>
So I introduced the component for you, It is time to see how it could work. <br/>
You would integrate this component in your main ViewModel as like this.

    function MainVM() {
        var self = this;

        self.PagingComponent = ko.observable(new Paging({
            pageSize: 10,      // how many items you would show in one page
            totalCount: 100,   // how many ALL the items do you have.
        }));

        self.currentPageSubscription = self.PagingComponent().CurrentPage.subscribe(function (newPage) {
            // here is the code which will be executed when the user change the page.
            // you can handle this in the way you need.
            // for example, in my case, I am requesting the data from the server again by making an ajax request
            // and then updating the component

            var data = /*bring data from server , for example*/
            self.PagingComponent().Update({

                // we need to set this again, why? because we could apply some other search criteria in the bringing data from the server, 
                // so the total count of all the items could change, and this will affect the paging
                TotalCount: data.TotalCount,

                // in most cases we will not change the PageSize after we bring data from the server
                // but the component allow us to do that.
                PageSize: self.PagingComponent().PageSize(),

                // use this statement for now as it is, or you have to made some modifications on the 'Update' function.
                CurrentPage: self.PagingComponent().CurrentPage(),
            });
        });

        self.dispose = function () {
            // you need to dispose the manual created subscription, you have created before.
            self.currentPageSubscription.dispose();
        }
    }

*Last but not least*, Sure do not forget to change the binding in the html component according to you special viewModel, or wrap all the component with the `with binding` like this

    <div data-bind="with: PagingComponent()">
        <!-- put the component here -->
    </div>
