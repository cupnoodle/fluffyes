# Show price in 2 decimal places using NumberFormatter

Ever want to show a price number nicely like "21.58" instead of "21.58223" after applying tax or discount?

We can turn 21.58223 to 21.58 by rounding up or down using [NumberFormatter](https://developer.apple.com/documentation/foundation/numberformatter) class as below : 

```swift

let formatter = NumberFormatter()
formatter.numberStyle = .decimal

// minimum decimal digit, eg: to display 2 as 2.00
formatter.minimumFractionDigits = 2

// maximum decimal digit, eg: to display 2.5021 as 2.50
formatter.maximumFractionDigits = 2

// round up 21.586 to 21.59. But doesn't round up 21.582, making it 21.58
formatter.roundingMode = .halfUp

let price = 21.58223
let roundedPriceString = formatter.string(for: price)

// output "rounded price is 21.58"
print("rounded price is \(roundedPriceString!)")

```
<br>

We will first create a NumberFormatter instance and configure its options as shown above.  

Then we will convert the original number into rounded output string using **.string(for: )** method, this will convert the input number into a string using the format options specified.

You can configure the **.roundingMode** to `.up`, `.down` , `.halfUp` and [etc](https://developer.apple.com/documentation/foundation/numberformatter.roundingmode) .

`.up` will add 1 to the maximum decimal digits if it has any decimal after the maxmium decimal digits, eg: 2.120001 to 2.13

`.down` will ignore and cut off the decimals after the maximum decimal digits, eg: 2.12999 to 2.12

`.halfUp` will add 1 to the maximum decimal digits if the decimals after the maximum decimal digits is more than half of the value. eg: 21.5864 to 21.59, 21.585 to 21.59, but 21.5821 to 21.58 


<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> No idea on how to start learning iOS Development? </h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/roadmapThumb.png" style="max-width: 200px;"></img>Get the <strong>Roadmap</strong> of learning iOS Development and learn iOS knowledge by hands-on coding of projects.
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/87821037/submissions" method="post" data-drip-embedded-form="87821037">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">Name<span style="color:#952B45;">*</span></label><br />
                    <input type="text" id="drip-firstname" name="fields[firstname]" value="" />
                </div>
                <div>
                    <label for="drip-email">Email Address<span style="color:#952B45;">*</span></label><br />
                    <input type="email" id="drip-email" name="fields[email]" value="" />
                </div>
              <div>
                <br>
                <input type="submit" value="Send me the Roadmap!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>