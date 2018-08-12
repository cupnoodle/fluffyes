# Replicating Spotify's Now Playing UI using Auto Layout - Part 2 / 2

In [part 1](https://fluffy.es/spotify-1/), we have laid out the basic layout of the album art collection view and playback buttons. In this part, we will implement the zooming effect / fading effect of the album art cell when scrolled. This post assume that you already knew about [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/) and [Delegate](https://fluffy.es/eli-5-delegate/).



## Set Data source and Delegate of Collection View

Create a custom collection view cell type, let's name it as "CoverCollectionViewCell". Set the class of the cell in Storyboard to "CoverCollectionViewCell". Then connect the album art image view in the cell in Storyboard to create an UIImageView outlet.

```swift
//  CoverCollectionViewCell.swift

class CoverCollectionViewCell: UICollectionViewCell {
	@IBOutlet weak var coverImageView: UIImageView!
}
```
<br>



Link the collection view into the view controller, and set data source and delegate (UICollectionViewDelegateFlowLayout).

```swift
class ViewController: UIViewController {
	@IBOutlet weak var coverCollectionView: UICollectionView!
    // in IB, click on the attribute inspector of the collection view cell, and find the 'Reuse Identifier'
	let cellReuseIdentifier = "cell"
    
    override func viewDidLoad() {
        coverCollectionView.dataSource = self
		coverCollectionView.delegate = self
        
        // hide the scroll indicator
		coverCollectionView.showsHorizontalScrollIndicator = false
    }
}

extension ViewController : UICollectionViewDataSource {
    // hardcode to show 10 cells, you can use array for this if you want
	func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
		return 10
    }
	
	func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
		let cell = collectionView.dequeueReusableCell(withReuseIdentifier: cellReuseIdentifier, for: indexPath) as! CoverCollectionViewCell
		
		return cell
	}
}

// Cell height is equal to the collection view's height
// Cell width = cell height = collection view's height
extension ViewController : UICollectionViewDelegateFlowLayout {
	func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
		
		return CGSize(width: collectionView.frame.size.height, height: collectionView.frame.size.height)
	}
}
```
<br>


Now we have a somewhat dynamic collection view, and the cell size are dynamic based on the collection view size / screen size!



## Add Insets to Collection View

Our collection view currently looks like this :
![Collection view no inset](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/noSpacing.png)



The first collection view cell is not center aligned as it sticks to the left edge of the collection view. Same goes to the last cell as it sticks to the right edge. We will add some padding to the left and right of the collection view, so that when user scroll to the first / last cells, there will be some space on the left / right edge.



```swift
override func viewDidLoad() {
    //....
    coverCollectionView.dataSource = self
    coverCollectionView.delegate = self
    coverCollectionView.showsHorizontalScrollIndicator = false
		
    // padding space = collection view width - cell width
    let leftPadding = (coverCollectionView.frame.size.width - coverCollectionView.frame.size.height) / 2.0
    let rightPadding = leftPadding
    
    coverCollectionView.contentInset = UIEdgeInsets(top: 0, left: leftPadding, bottom: 0, right: rightPadding)
}
```
<br>



Remember the aspect ratio we set for the collection view in previous part? The cell height is equal to the collection view's height, cell width is equal to cell height, hence cell width is equal to the collection's view height. If you are confused about how the value of padding is calculated, here's some explanation : 



```
left padding = right padding

cell height = collection view height
cell width = cell height
cell width = collection view height

left padding + right padding = collection view width - cell width
left padding + right padding = collection view width - cell height
left padding + right padding = collection view width - collection view height
left padding = (collection view width - collection view height) / 2
right padding = (collection view width - collection view height) / 2
```

<br>



After setting the left padding and right padding insets, when scrolling to the first / last cell, it looks good now :  

![left spacing](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/leftSpacing.png)



![right spacing](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/rightSpacing.png)



## Add Zoom Effect to the Collection View Cell when scrolled

The fun part! When scrolling the collection view, as the middle cell is being scrolled to the side, its size become smaller, and the neighbouring cell approaching middle will become larger.



One way to think about this is that the size of the cell become smaller when it become further from the center of the collection view. When the cell is in the center, it has its original size, meaning the scale is 1x. When the cell move to left / right, we make it smaller by multiplying its size with a smaller scale (eg: 0.9x). The value of scale is determined by the distance from the center of the cell to the center of the collection view.



![distance visualization](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/distance.png)



```
// abs means absolute value, eg: abs(-5) = 5, abs(5) = 5
distance = abs(cell center X - collection view center X)

// 1.0 is the max scale, which is the value when the cell is in the exact center
scale = 1.0 - (distance / collection view center X)

// maximum cell size is the value returned from sizeForItemAt: method
cell size = maximum cell size * scale
```



If we used the above formula, we will get the scaling effect but the scale can become zero or less than zero when the distance from cell center to collection view center is larger than the collection view center X position value!

![scale Smol](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/scaleSmol.gif)



To solve this problem, we will multiply value of `(distance / collection view center x)` with a number smaller than 1 so that it doesn't get too big, making `scale` too small. 



Let's multiply it with 0.105 (you can use any other number smaller than 1 if you like), we will modify the above calculation to this : 

```
// abs means absolute value, eg: abs(-5) = 5, abs(5) = 5
distance = abs(cell center X - collection view center X)

// 1.0 is the max scale, which is the value when the cell is in the exact center
// multiply the (distance / collection view center X) with 0.105 to make it smaller
scale = 1.0 - ((distance / collection view center X) * 0.105)

// maximum cell size is the value returned from sizeForItemAt: method
cell size = maximum cell size * scale
```

<br>



It looks better now: 
![scale Good](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/scaleGood.gif)



If you pay attention to the Spotify album art collection view, the album art doesn't immediately get shrinked  when it is moving away from the center. There is a minimum scroll distance required before the album art get shrinked.



Notice that within a certain scrolling distance, the size of the album art doesn't change :
![scroll fixed](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/scaleFixed.gif)



We can implement the minimum scroll distance by adding a tolerance value and a if-check like this: 

```
// abs means absolute value, eg: abs(-5) = 5, abs(5) = 5
distance = abs(cell center X - collection view center X)

// 1.0 is the max scale, which is the value when the cell is in the exact center
tolerance = 0.02
scale = 1.0 + tolerance - ((distance / collection view center X) * 0.105)

if(scale > 1.0){
  scale = 1.0
}

// maximum cell size is the value returned from sizeForItemAt: method
cell size = maximum cell size * scale
```



What this do is that it add 0.02 to the scale value, for example, let say on a certain scrolling distance, the scale is supposed to be 0.98, then a 0.02 is added to it, making the scale 1.0, thus on that distance, the album art still haven't shrink. When the album art is on the exact middle, the scale value would be 1.02, we then update it to 1.0 by using the if check so that it doesn't grow larger than 1.0. 



Now it looks good, we can apply the calculation into code like this :

```swift
extension ViewController : UIScrollViewDelegate {
    // perform scaling whenever the collection view is being scrolled
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        
        // center X of collection View
        let centerX = self.coverCollectionView.center.x
	
        // only perform the scaling on cells that are visible on screen
        for cell in self.coverCollectionView.visibleCells {
			
            // coordinate of the cell in the viewcontroller's root view coordinate space
            let basePosition = cell.convert(CGPoint.zero, to: self.view)
            let cellCenterX = basePosition.x + self.coverCollectionView.frame.size.height / 2.0
            
            let distance = fabs(cellCenterX - centerX)
			
            let tolerance : CGFloat = 0.02
            var scale = 1.00 + tolerance - (( distance / centerX ) * 0.105)
            if(scale > 1.0){
                scale = 1.0
            }
			
            // set minimum scale so the previous and next album art will have the same size
            // I got this value from trial and error
            // I have no idea why the previous and next album art will not be same size when this is not set 😅
            if(scale < 0.860091){
                scale = 0.860091
            }
			
			// Transform the cell size based on the scale
            cell.transform = CGAffineTransform(scaleX: scale, y: scale)
		}
	}
}
```

<br>

Since collection view is a subclass of scrollview, when you set `collectionView.delegate = self` in the viewDidLoad, you can use the scroll view delegate method on the collection view in the view controller.



## Add Fade effect to the Collection View Cell when scrolled

Similar to the zoom effect, we want to dim the album art when it is being scrolled to the side. Since we already have the scale value, we can use it to indicate the alpha of the album art image view. (Alpha 1.0 is fully opaque, alpha 0.0 is fully invisible)



We can add the code to change the alpha of the image view after the `cell.transform = CGAffineTransform` line.



```swift
extension ViewController : UIScrollViewDelegate {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let centerX = self.coverCollectionView.center.x
	
		for cell in self.coverCollectionView.visibleCells {
            //....
            cell.transform = CGAffineTransform(scaleX: scale, y: scale)
            
            // change the alpha of the image view 
            let coverCell = cell as! CoverCollectionViewCell
            coverCell.coverImageView.alpha = scale
        }
    }
}
```
<br>




Since the value of **scale** is between 0.86 and 1.0 , using this as alpha value doesn't show much difference in transparency. We want the alpha value to between 0.25 and 1.0 . To transform the value range from **0.86 - 1.0** to **0.25 - 1.0**, we can write a function to transform the range using some [mathematical formula](https://stackoverflow.com/questions/5294955/how-to-scale-down-a-range-of-numbers-with-a-known-min-and-max-value).

```swift
extension ViewController : UIScrollViewDelegate {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let centerX = self.coverCollectionView.center.x
	
		for cell in self.coverCollectionView.visibleCells {
            //....
            cell.transform = CGAffineTransform(scaleX: scale, y: scale)
            
            // change the alpha of the image view 
            let coverCell = cell as! CoverCollectionViewCell
            coverCell.coverImageView.alpha = changeSizeScaleToAlphaScale(scale)
        }
    }
    
    // map the scale of cell size to alpha of image view using formula below
    // https://stackoverflow.com/questions/5294955/how-to-scale-down-a-range-of-numbers-with-a-known-min-and-max-value
    func changeSizeScaleToAlphaScale(_ x : CGFloat) -> CGFloat {
        let minScale : CGFloat = 0.86
        let maxScale : CGFloat = 1.0
		
        let minAlpha : CGFloat = 0.25
        let maxAlpha : CGFloat = 1.0
		
        return ((maxAlpha - minAlpha) * (x - minScale)) / (maxScale - minScale) + minAlpha
	}
}
```
<br>



The album art collection view is looking good now! When an album art is moving towards the edge of screen, it will get dimmed.
![alpha](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/alpha.gif)



The only thing left now is a "snap-to nearest" effect, meaning if user stop scrolling the collection view, it should snap to the nearest album art and putting it in the middle.



## Add Snap-to Nearest effect when user stops scrolling

The logic of snap-to nearest is that when user stops scrolling, the collection view should auto scroll to the current largest sized cell, and put the cell in the middle. The method that will be called when user stop scrolling a scrollview is `scrollViewDidEndDragging `. 



To find the largest sized cell, we will loop through all the current visible cells in the collection view, record down the largest width value and the index of the cell with the largest width value. Then we will call `collectionView.scrollToItem(at: )`  to instruct the collection view to scroll to that cell.



```swift
extension ViewController : UIScrollViewDelegate {
	func scrollViewDidScroll(_ scrollView: UIScrollView) {
        //....
    }
    
    // for custom snap-to paging, when user stop scrolling
    func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {
        return
        var indexOfCellWithLargestWidth = 0
        var largestWidth : CGFloat = 1
        
        for cell in self.coverCollectionView.visibleCells {
            if cell.frame.size.width > largestWidth {
                largestWidth = cell.frame.size.width
                if let indexPath = self.coverCollectionView.indexPath(for: cell) {
                    indexOfCellWithLargestWidth = indexPath.item
                }
            }
        }
        
        coverCollectionView.scrollToItem(at: IndexPath(item: indexOfCellWithLargestWidth, section: 0), at: .centeredHorizontally, animated: true)
    }
}
```

<br>



After adding the snap-to nearest effect, it will look like this when user releases finger during scroll:
![snap to](https://iosimage.s3.amazonaws.com/2018/25-spotify-2/snapTo.gif)



Congratulation if you have made it this far, you have successfully replicated Spotify's Now Playing UI screen!

