## Determine which button got tapped inside UITableViewCell without using tag



Top and accepted answer : https://stackoverflow.com/questions/20655060/get-button-click-inside-uitableviewcell


Tag is original intended to identify a view quickly , not to use as a data store (in this case, the row / index of the item). Abusing tag can quickly lead to a nightmare like this <http://doing-it-wrong.mikeweller.com/2012/08/youre-doing-it-wrong-4-uiview.html>



use closure or delegate 