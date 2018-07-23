# SimpleBarcodeScanner [![Android Arsenal]( https://img.shields.io/badge/Android%20Arsenal-SimpleBarcodeScanner-green.svg?style=flat )]( https://android-arsenal.com/details/1/7004 ) [![](https://img.shields.io/badge/minSdk-15-brightgreen.svg)](https://github.com/bobekos/SimpleBarcodeScanner)

Barcode Scanner by Google Mobile Vision Api with RxJava

## Getting Started

### Setting up the dependency

The first step is to include SimpleBarcodeScanner into your project, as a Gradle dependency:
```
Add it in your root build.gradle at the end of repositories:

allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```
[![](https://jitpack.io/v/bobekos/SimpleBarcodeScanner.svg)](https://jitpack.io/#bobekos/SimpleBarcodeScanner)
```
Add the dependency:

implementation 'com.github.bobekos:SimpleBarcodeScanner:x.x.xx'
```

## Usage

### Include following code in your layout:

```xml
<com.bobekos.bobek.scanner.BarcodeView
        android:id="@+id/barcodeView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
```

For all supported attributes see the list below (not supported yet)

### Include followin code in your activity or fragment

```kotlin
class MainActivity : AppCompatActivity() {

    private var mDisposable: Disposable? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }

    override fun onStart() {
        super.onStart()

        //make sure to request camera permission before the subscription

        mDisposable = barcodeView
                .getObservable()
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(
                        { barcode ->
                            //handle barcode object
                        },
                        { throwable ->
                            //handle exceptions like no available camera for selected facing
                        })
    }

    override fun onStop() {
        super.onStop()

        mDisposable?.dispose()
    }
}
```

## Features

### Available Options

```
.setBarcodeFormats(Barcode.QR_CODE)
```

Which barcode format should be detected. Default value is all formats.

```
.setFacing(CameraSource.CAMERA_FACING_BACK)
```

Set the camera facing. Default value is back facing.

```
.setFlash(false)
```

Turn on the flash. Default value is false. (Also changeable after the subscription)

```
.setAutoFocus(true)
```

Enable autofocus. Default value is true.

```
.setPreviewSize(width, height)
```

Set preview size for the camera source. The given preview size is calculated to the closet value from camera available sizes. Default values are the display dimensions.

```
.drawOverlay()
```

Draw a overlay view over the detected barcode. Default overlay is a white rect.

```
.setBeepSound(true)
```

Play Beep sound at barcode detection. Default value is true. (Also changeable after the subscription)

```
.setVibration(500L)
```

Vibrate at barcode detection. Default value is 500ms. (Also changeable after the subscription)

### Advanced Options

#### Custom overlay

There a already two implemented overlay views (BarcodeRectOverlay and BarcodeTextOverlay). 
To create your own overlay, you only need to implement the "BarcodeOverlay" interface to your custom view. 
The method "onUpdate" passes the position of the barcode on the screen and its value. An own view looks like this, for example: 

```kotlin
class RedRectOverlay : View, BarcodeOverlay {

    //constructors

    private lateinit var rect: Rect

    private val paint by lazy {
        Paint().apply {
            color = Color.RED
            style = Paint.Style.STROKE
            strokeWidth = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 3f, context?.resources?.displayMetrics)
        }
    }

    init {
        setWillNotDraw(false)
    }

    override fun onUpdate(posRect: Rect, barcodeValue: String) {
        rect = posRect
        invalidate()
    }

    override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)

        if (::rect.isInitialized) {
            canvas?.drawRect(rect, paint)
        }
    }
}

// Activity or Fragment
barcodeView
	.drawOverlay(RedRectOverlay(this))
```

If the barcode detection failed or finished the "onUpdate" method passed empty values for the position and barcode value.

#### Rx

You have full control of the observable which is returned from the BarcodeView.
Nevertheless, I have prepared a few examples to show what is possible.

```
//filter the detected results

.getObservable()
.filter { barcode ->
    barcode.displayValue == "12345"
}
.observeOn...
```

```
//skip results until the raw value changed

.getObservable()
.distinctUntilChanged { barcode1, barcode2 ->
	barcode1.rawValue == barcode2.rawValue
}
.observeOn...
```

```
//get only first item

.getObservable()
.firstOrError()
.observeOn...
```

```
// combine your api/database/etc. observables directly

.getObservable()
.firstOrError()
.flatMap { barcode ->
    ApiService.getDateByBarcodeId(barcode.displayValue)
}
.flatMap ...
```

and many more...

**On which thread does the detection run?**

The detection runs on an background thread. Don't forget to set the correct scheduler to the "observeOn" method of the observable
if you want to have the result on the main android thread for example.

**It is necessary to dispose the subscription on his own?**

Short answer 'yes'. Although the observable called "onComplete" when the surface is destroyed. 
But to avoid memory leaks you should always dispose your subscription.

## Resources and Credits

* [google android-vision sample](https://github.com/googlesamples/android-vision/tree/master/visionSamples/barcode-reader)
* [rxJava](https://github.com/ReactiveX/RxJava)
* [rxAndroid](https://github.com/ReactiveX/RxAndroid)

## License

    Copyright 2018 Bobek Bobekos

    Licensed under the Apache License, Version 2.0 (the "License"); 
    you may not use this file except in compliance with the License. 
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0
   
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and 
    limitations under the License.
