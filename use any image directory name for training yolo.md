# use any image directory name for training yolo 



By default, you can only set the name of image directory as 'JPEGImage' or 'images' for training yolo model. To

solve this problem, we should change the code at 'src/data.c', as shown below.

```c
#include<libgen.h>
...

void fill_truth_detection(char *path, int num_boxes, float *truth, int classes, int flip, float dx, float dy, float sx, float sy)
{    
    char *buffer;								// defined a buffer to save image path
    buffer = strdup(path);

    char *image_path, *image_folder;			 // get the name of the image directory
    image_path = dirname(buffer);
    image_folder = basename(image_path);
    
    char labelpath[4096];
    
    find_replace(path, "images", "labels", labelpath);
    find_replace(labelpath, "JPEGImages", "labels", labelpath);
    
    // replace the image directory name in the path
    find_replace(labelpath, image_folder, "labels", labelpath);
    
    find_replace(labelpath, "raw", "labels", labelpath);
    find_replace(labelpath, ".jpg", ".txt", labelpath);
    find_replace(labelpath, ".png", ".txt", labelpath);
    find_replace(labelpath, ".JPG", ".txt", labelpath);
    find_replace(labelpath, ".JPEG", ".txt", labelpath);
    ...
    }
```

 