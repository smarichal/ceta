# CETA - Mixed-reality environment for low cost Android tablets and 3D printed hardware
This repository contains all the necessary resources to reproduce the CETA mixed reality environment from scratch. 

Here you can find:
* Hardware design: 3D design for 3D printers and laser cutters: https://github.com/smarichal/ceta/tree/master/hardware-design
* Software implementation: 
    1) computer vision module to detect the augmented rods (https://github.com/smarichal/ceta-vision-core-library.git)
    2) Android CETA game (https://github.com/ewelinka/CETA.git) 


If you have any questions about the development of the game, vision module or hardware design don't hesitate to reach us at: 

sebastian.marichal@upf.edu (vision module and general issues)
ebakala@fing.edu.uy (Android CETA Game)
gustavo.sansone@gmail.com  (Hardware design)

# Layers

The architecture of CETA has 3 layers: 
Layer 1 - Android activity/game: Just an Android app. The only tricky part is that in this layer we capture the frames from the camera and convert them to RGBA. This is implemented in AndroidLauncher.java. Then, using the captured frame, the CVBlocksManager invokes the detection algorithm and obtaining the high level representation of the augmented rods (Block.java). An important point is that The CVBlocksManager calls the detection algorithm asynchronously: 

    @Override
    public void updateDetected() {
        new Thread(new Runnable() {
            public void run() {
                Mat frame = ((CetaGame)game).getAndBlockLastFrame();
                Core.flip(frame, frame, 0);
                final Set<Block> finalSet = topCodeDetector.detectBlocks(frame);
                detectionReady = true;
                ((CetaGame) game).releaseFrame();

                Gdx.app.postRunnable(new Runnable() {
                    @Override
                    public void run() {
                        results.clear();
                        results.add(finalSet);
                    }
                });
            }
        }).start();
    }
    
Layer 2 - Augmented rods detection module: This layer interpretates the composition of topcode markers as augmented rods. Here we also implemented a cache algorithm to support partial occlusions of the blocks. This means that, in general, just one topcode  marker visible is enough to detect the whole block querying the cache information. This layer is implemented in TopCodeDetector.java which uses the Scanner.java (provided by TopCode Library) and SpotsCache.java.

Layer 3 - Low level vision layer, i.e., TopCode library. Process the frame and detect the markers. The code at this level could be changed in order to detect objects using color, shape, or other features. However, this will probably impact on 
the layer 2 since they are highly connected.

# Git Modules
Beyond the logical division in 3 layers, the source code is divided in 2 main git modules, CETA and ceta-vision-core-library. The former containts the Android activity (top level layer) while the later contains the layer 2 and 3, i.e., the augmented rods detection module and the TopCode vision library. 

# How to create tangible rods
![Sample of augmented rod](https://github.com/smarichal/ceta/blob/master/hardware-design/blocks/augmentedrod.png)
Constraints for blocks design: 

1. Equal distances between markers within a block, i.e.,d1=d2=d3. Let’s call this distance d.
2. The distance d must be the same in all the blocks.
3. Just one marker id can be used in each block. This is M1=M2=M3, let’s call this marker M.
4. The marker must be mapped with the value of the block, i.e., in this case M has to be mapped with
blocks representing number 3. You can modify this mapping in the class BlocksMarkersMap.java: 
    
	    public static int[] block1 = {31, 61, 103, 179, 227, 271, 283, 355, 391, 453};
	    public static int[] block2 = {93, 117, 185, 203, 793}; 
	    public static int[] block3 = {563, 651, 361, 309};
	    public static int[] block4 = {171, 555, 421};
	    public static int[] block5 = {1173, 1189, 677};
	
5. Background color could be changed. However, the TopCode vision algorithm performs better with higher
contrast. The best scenario is black rings and white background.

We are provinding a blocks layout sample file ready to print and place on top of the physical block:
![Blocks layout](https://github.com/smarichal/ceta/blob/master/hardware-design/blocks/markers.png)

# OpenCV
OpenCV is included in the project and linked statically in order to not have to install OpenCV Manager in the target device.
The following code belongs to AndroidLauncher.java: 

    static {
        if (!OpenCVLoader.initDebug()) {
            // Handle initialization error
        	Log.e(TAG,"Severe error, openCV could not be loaded!");
        }else{
	    	Log.i(TAG, "OpenCV loaded successfully");
        }
	}

# Clone and compile the project using Eclipse
1- git clone https://github.com/smarichal/ceta.git
2- cd ceta/CETA
3- git submodule update --recursive --remote
4- Import maven project from eclipse menu: 

![Import maven project](https://github.com/smarichal/ceta/blob/master/documentation/importMavenProject.png)
![Import maven project](https://github.com/smarichal/ceta/blob/master/documentation/ImportMavenProject2.png)
5- At the beginning you will see the project doesn't compile, this is because we need to import the OpenCV library as a project. 
Download the android version of OpenCV and add it as a project, it should looks like this: 
![OpenCV project imported](https://github.com/smarichal/ceta/blob/master/documentation/ImportOpenCVProject.png)

6- Check that the ceta-vision-core-library-android project is being compiled into a .jar instead of an .apk. 
![Ceta vision core compiled as .jar](https://github.com/smarichal/ceta/blob/master/documentation/ceta-vision-core-library-androidJar.png)

7- Copy and paste the ceta-vision-core-library-android.jar into the bin folder of the CETA project: 
![jar into bin folder](https://github.com/smarichal/ceta/blob/master/documentation/pasteJarIntoBinFolder.png)

8 - Add the ceta-vision-core-library-android.jar to the build path of the CETA project
![add vision core to build path](https://github.com/smarichal/ceta/blob/master/documentation/AddJarToBuildPath.png)

9- Create a User library with the opencv project. Let's call it opencv-248. Also, add openCV jar to the library
![Create user library](https://github.com/smarichal/ceta/blob/master/documentation/createUserLibrary1.png)
![Add jar to the library](https://github.com/smarichal/ceta/blob/master/documentation/createUserLibrary2-addOpenCVJar.png)

10- add the opencv-248 library to CETA-android build path.  Add also a reference to this library.

![adding library to buildpath](https://github.com/smarichal/ceta/blob/master/documentation/addUserLibraryToCETABuildPath.png)

![adding reference to opencv project](https://github.com/smarichal/ceta/blob/master/documentation/addReferenceToOpenCVLibrary.png)

![export libraries](https://github.com/smarichal/ceta/blob/master/documentation/orderAndExportAddedLibraries.png)

Now you might have the whole project compiling and ready to be deployed in your Android device. If you are having any issues please contact us and we will be happy to help you.
Thank you!!
