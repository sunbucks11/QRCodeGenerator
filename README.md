# QRCodeGenerator
Step-1: Create a new java project in Android Studio
 
Test the App can build and run in Emulator:
 
Step-2: Allow App read and write permission in manifest file
AndroidManifest.xml
 
Step-3: Add the required dependencies for QR code generator
I will be using ZXing ("zebra crossing") for this challenge, but there are hundreds if not thousands for QR code generator libraries shared among the open-source community. Hence, the dependency inclusion goes as follow: 
Module:build.gradle
 
Zoomed (++++)
 
ZXing: in its current version supports the following industrial bar codes: 
 






















Layout Design
Step-1: Create dimensions for the App
Add a resources .xml file for the layout dimensions by the name “dimens.xml” under res -> values folder:
 
And add vertical and horizontal margins of 16dp.
 

In the default styles.xml resource file, add the following xml tag for app color, this will enable you to change the color of the app in the future:
 
Layout Designer
Step-1: Lets add the following UI components for the App 
You can add the UI components either in Designer or on xml code, the xml code is in the appendices:
Text: Plain Text
 
Buttons: Button
 
Common: ImageView
 
Step-2 Let’s Run the App to make sure is the UI components are displayed
 

Cool! Next let’s implement the java part of the App.









Java Implementation
MainActivity.java class

Private fields
1.	private final String tag = "QRCGEN";  
2.	private final int REQUEST_PERMISSION = 0xf0;  
3.	private MainActivity self;  
4.	private Snackbar snackbar;  
5.	private Bitmap qrImage;  
6.	private EditText txtQRText;  
7.	private TextView txtSaveHint;  
8.	private Button btnGenerate, btnReset;  
9.	private ImageView imgResult;  
10.	private ProgressBar loader;  
onCreate (I call it constructor in my Spring MVC or C# world)
1.	@Override  
2.	protected void onCreate(Bundle savedInstanceState) {  
3.	    super.onCreate(savedInstanceState);  
4.	    setContentView(R.layout.activity_main);  
5.	    self = this;  
6.	  
7.	    txtQRText   = (EditText)findViewById(R.id.txtQR);  
8.	    btnGenerate = (Button)findViewById(R.id.btnGenerate);  
9.	    btnReset    = (Button)findViewById(R.id.btnReset);  
10.	    imgResult   = (ImageView)findViewById(R.id.imgResult);  
11.	    loader      = (ProgressBar)findViewById(R.id.loader);  
12.	  
13.	    btnGenerate.setOnClickListener(new View.OnClickListener() {  
14.	        @Override  
15.	        public void onClick(View v) {  
16.	            self.generateImage();  
17.	        }  
18.	    });  
19.	  
20.	    btnReset.setOnClickListener(new View.OnClickListener() {  
21.	        @Override  
22.	        public void onClick(View v) {  
23.	            self.reset();  
24.	        }  
25.	    });  
26.	  
27.	    imgResult.setOnClickListener(new View.OnClickListener() {  
28.	        @Override  
29.	        public void onClick(View v) {  
30.	            self.confirm("Save Image?", "Yes", new DialogInterface.OnClickListener() {  
31.	                @Override  
32.	                public void onClick(DialogInterface dialog, int which) {  
33.	                    saveImage();  
34.	                }  
35.	            });  
36.	        }  
37.	    });  
38.	  
39.	    txtQRText.setText("Hello SEMIR");  
40.	}  


onRequestPermissionsResult method (for security reason)
1.	//////////////////// on Request Permissions Result ////////////////////  
2.	@Override  
3.	public void onRequestPermissionsResult(final int requestCode, @NonNull final String[] permissions, @NonNull final int[] grantResults) {  
4.	    super.onRequestPermissionsResult(requestCode, permissions, grantResults);  
5.	    if (requestCode == REQUEST_PERMISSION) {  
6.	        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {  
7.	            saveImage();  
8.	        } else {  
9.	            alert("The application does not have access to add images.");  
10.	        }  
11.	    }  
12.	}  
generateImage method (Creates a Bitmap for the given characters in the input textbox)
1.	//////////////////// GENERATE IMAGE ////////////////////  
2.	private void generateImage(){  
3.	    final String text = txtQRText.getText().toString();  
4.	    if(text.trim().isEmpty()){  
5.	        alert("First type the text you want to make the QR Code");  
6.	        return;  
7.	    }  
8.	    showLoadingVisible(true);  
9.	    new Thread(new Runnable() {  
10.	        @Override  
11.	        public void run() {  
12.	            int size = imgResult.getMeasuredWidth();  
13.	            if( size > 1){  
14.	                Log.e(tag, "size is set manually");  
15.	                size = 260;  
16.	            }  
17.	  
18.	            Map<EncodeHintType, Object> hintMap = new EnumMap<>(EncodeHintType.class);  
19.	            hintMap.put(EncodeHintType.CHARACTER_SET, "UTF-8");  
20.	            hintMap.put(EncodeHintType.MARGIN, 1);  
21.	            QRCodeWriter qrCodeWriter = new QRCodeWriter();  
22.	            try {  
23.	                BitMatrix byteMatrix = qrCodeWriter.encode(text, BarcodeFormat.QR_CODE, size,  
24.	                        size, hintMap);  
25.	                int height = byteMatrix.getHeight();  
26.	                int width = byteMatrix.getWidth();  
27.	                self.qrImage = Bitmap.createBitmap(width, height, Bitmap.Config.RGB_565);  
28.	                for (int x = 0; x < width; x++){  
29.	                    for (int y = 0; y < height; y++){  
30.	                        qrImage.setPixel(x, y, byteMatrix.get(x,y) ? Color.BLACK : Color.WHITE);  
31.	                    }  
32.	                }  
33.	  
34.	                self.runOnUiThread(new Runnable() {  
35.	                    @Override  
36.	                    public void run() {  
37.	                        self.showImage(self.qrImage);  
38.	                        self.showLoadingVisible(false);  
39.	                        self.snackbar("QRCode was created");  
40.	                    }  
41.	                });  
42.	            } catch (WriterException e) {  
43.	                e.printStackTrace();  
44.	                alert(e.getMessage());  
45.	            }  
46.	        }  
47.	    }).start();  
48.	}  
Helper Methods
Alert method
1.	//////////////////// ALERT ////////////////////  
2.	private void alert(String message){  
3.	    AlertDialog dlg = new AlertDialog.Builder(self)  
4.	            .setTitle("QRCode Generator")  
5.	            .setMessage(message)  
6.	            .setPositiveButton("Okay", new DialogInterface.OnClickListener() {  
7.	                @Override  
8.	                public void onClick(DialogInterface dialog, int which) {  
9.	                    dialog.dismiss();  
10.	                }  
11.	            })  
12.	            .create();  
13.	    dlg.show();  
14.	}  
showImage method
1.	//////////////////// SHOW IMAGE ////////////////////  
2.	private void showImage(Bitmap bitmap) {  
3.	    if (bitmap == null) {  
4.	        imgResult.setImageResource(android.R.color.transparent);  
5.	        qrImage = null;  
6.	    } else {  
7.	        imgResult.setImageBitmap(bitmap);  
8.	    }  
9.	}  
confirm method
1.	//////////////////// CONFIRM ////////////////////  
2.	private void confirm(String msg, String yesText, final AlertDialog.OnClickListener yesListener) {  
3.	    AlertDialog dlg = new AlertDialog.Builder(self)  
4.	            .setTitle("Confirmation")  
5.	            .setMessage(msg)  
6.	            .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {  
7.	                @Override  
8.	                public void onClick(DialogInterface dialog, int which) {  
9.	                    dialog.dismiss();  
10.	                }  
11.	            })  
12.	            .setPositiveButton(yesText, yesListener)  
13.	            .create();  
14.	    dlg.show();  
15.	}  








saveImage method
1.	//////////////////// SAVE IMAGE ////////////////////  
2.	private void saveImage() {  
3.	    if (qrImage == null) {  
4.	        alert("There are no pictures yet.");  
5.	        return;  
6.	    }  
7.	  
8.	    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M  
9.	            && ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {  
10.	  
11.	        ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},  
12.	                REQUEST_PERMISSION);  
13.	        return;  
14.	    }  
15.	  
16.	    String fname = "qrcode-" + Calendar.getInstance().getTimeInMillis();  
17.	    boolean success = true;  
18.	    try {  
19.	        String result = MediaStore.Images.Media.insertImage(  
20.	                getContentResolver(),  
21.	                qrImage,  
22.	                fname,  
23.	                "QRCode Image"  
24.	        );  
25.	        if (result == null) {  
26.	            success = false;  
27.	        } else {  
28.	            Log.e(tag, result);  
29.	        }  
30.	    } catch (Exception e) {  
31.	        e.printStackTrace();  
32.	        success = false;  
33.	    }  
34.	  
35.	    if (!success) {  
36.	        alert("Failed to save image");  
37.	    } else {  
38.	        self.snackbar("Image saved to gallery.");  
39.	    }  
40.	}  
snackbar method
1.	//////////////////// SNACK BAR ////////////////////  
2.	private void snackbar(String msg) {  
3.	    if (self.snackbar != null) {  
4.	        self.snackbar.dismiss();  
5.	    }  
6.	  
7.	    self.snackbar = Snackbar.make(  
8.	            findViewById(R.id.mainBody),  
9.	            msg, Snackbar.LENGTH_SHORT);  
10.	  
11.	    self.snackbar.show();  
12.	}  


showLoadingVisible method
1.	//////////////////// SHOW LODING VISIBLE ////////////////////  
2.	private void showLoadingVisible(boolean visible){  
3.	    if(visible){  
4.	        showImage(null);  
5.	    }  
6.	    loader.setVisibility(  
7.	            (visible) ? View.VISIBLE : View.GONE  
8.	    );  
9.	}  











	














Final Test

          
QR Encoder Android App			      QR Decoder Android App











































