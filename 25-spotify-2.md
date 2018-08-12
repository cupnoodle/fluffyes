# Replicating Spotify's Now Playing UI using Auto Layout - Part 2 / 2

In [part 1](https://fluffy.es/spotify-1/), we have laid out the basic layout of the album art collection view and playback buttons. In this part, we will add the zooming effect / fading effect of the album art cell when scrolled.



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
scale = 1.0 - ((distance / collection view center X) * 0.105)

// maximum cell size is the value returned from sizeForItemAt: method
cell size = maximum cell size * scale
```

<br>



