# How to implement dynamic height table view cell (self sizing)



It's common to implement table view which contain cells with different length of text, as the content are retrieved from web APIs.



When implementing a dynamic height tableview cell, one common issue we get is that the text either got clipped or two elements get overlapped :

![clipped](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/clipped.png)





A quick search on google on 'how to make a dynamic height table view cell' will tell you to put in this two line :

```swift
tableView.rowHeight = UITableView.automaticDimension
tableView.estimatedRowHeight = 80
```



Putting these two lines seems easy, but somehow the cell refuse to expand to the right height even when the label exceed one line, and before you know it you have already spent two hours wrangling with the constraints inside the cell and still not working.



The key to making this work is actually setting enough constraints for the elements inside the cell so that **Auto layout knows how to calculate the height of the cell**.



To let Auto Layout know how to calculate the height of the cell, we need to **create sufficient constraints from top of the cell's content view to the bottom**.



![heightConstraints](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/heightConstraint.png)



For the cell above with two labels, we need to define the vertical spacing constraint from author name label to the cell content view top (**A**), the vertical spacing between author name label and quote label (**C**) and bottom constraint from quote label to the cell content view bottom (**E**).



**A** :

![constraint to content view](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/constraintContentView.png)



![constraintContentView2](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/constraintContentView2.png)

**C** :

![constraint between two labels](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/constraintContentView3.png)

![constraints between two labels](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/constraintContentView4.png)

**E** :

![constraint of label to bottom of content view](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/constraintContentView5.png)

![constraint of label to bottom of content view](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/constraintContentView6.png)





As for the height of author name label (**B**) and height of quote label (**D**), we do not need to set a height constraint for them as UILabel has [intrinsic content size](https://fluffy.es/what-is-intrinsic-content-size/). UILabel has intrinsic content size, this means that its width and height can be derived from the text string of the label and the font type and font size used.



As long as Auto Layout is able to calculate the height of the cell from top to bottom, it will show dynamic height cell nicely.



The previous overlapping example happens because I didn't define the vertical spacing constraint between author name label and quote label (**C**) :

![height constraint incomplete](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/heightConstraintIncomplete.png)

![clipped](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/clipped.png)



This makes Auto Layout unable to calculate the height of the cell.



When Auto Layout is unable to calculate the height of the cell due to insufficient constraint, it will use the default row height or the row height you have defined in Interface Builder, or the height derived from heightForRowAt function.



If we add another image view in the middle, then here's the vertical constraints we need to set (colored in Red, A, C, D, E and G) :

![image view](https://iosimage.s3.amazonaws.com/2020/70-self-sizing-cell/threeElements.png)



Image view without an image selected has no intrinsic content size, hence we need to set an explicit height constraint for it.



The key to achieve dynamic height table view cell is to **let Auto Layout know how to calculate the height of the cell**.



This article is an excerpt from the book [Making Sense of Auto Layout](http://autolayout.fluffy.es/?ref=scroll) , if you feel like you have no idea what you are doing when adding / removing / editing constraints,  give the sample chapter a try!



CTA of sample dynamic height project