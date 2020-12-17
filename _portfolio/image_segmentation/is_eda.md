---
layout: default
title: Image Segmentation - Exploratory Data Analysis
permalink: /is_eda/
---

### [Previous: Data Wrangling](https://cemalec.github.io/is_data_wrangling/)|[Next: Statistical Inference](https://cemalec.github.io/is_statistical_inference/)

## Questions to answer:

1) What is the average mitochondria area?

2) What is the average mitochondria volume?

3) Are the cells spherical, elliptical, some other odd shape?

4) How much variation is there in contrast?

5) Are there significant differences in average intensity in the two regions?


```python
#Import necessary packages and set plot types to allow interactive plots
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
%matplotlib notebook
```


```python
#Load numpy arrays from last notebook
training_inputs = np.load('training_inputs.npy')
training_ground_truth = np.load('training_ground_truth.npy')
```


```python
#A useful function I adapted from CS231n code
def visualize_grid(image_stack, upper_bound = 255.0, padding = 1):
    """
    Reshape a stack of images into a grid for easy visualization.
    
    Inputs:
    - image_stack: Data in shape (N, C, W, H)
    - upper_bound: pixel intensity will be scaled to the range [0, upper_bound]
    - padding: Number of blank pixels between elements of grid
    """
    
    N, C, W, H = image_stack.shape
    image_stack = image_stack.transpose(0,3,2,1)
    grid_size = int(np.ceil(np.sqrt(N)))
    grid_height = H * grid_size + padding * (grid_size - 1)
    grid_width = W * grid_size + padding * (grid_size - 1)
    grid = np.zeros((grid_height, grid_width, C))
    next_img = 0
    y0, y1 = 0, H
    for row in range(grid_size):
        x0, x1 = 0, W
        for col in range(grid_size):
            if next_img < N:
                img = image_stack[next_img]
                low, high = np.min(img), np.max(img)
                grid[y0:y1, x0:x1] = upper_bound * (img - low)/(high - low)
                next_img += 1
            x0 += W + padding
            x1 += W + padding
        y0 += H + padding
        y1 += H + padding
        
    return grid
```


```python
#Overlay images to highlight mitochondria, there may be a better way to do this since the black tiles also get overlaid
grid_image = visualize_grid(training_inputs[::11],padding = 50)
grid_overlay = visualize_grid(training_ground_truth[::11],padding = 50)
plt.imshow(grid_image[:,:,0].T.astype('uint8'), cmap = 'gray')
plt.imshow(grid_overlay[:,:,0].T.astype('uint8'), cmap = 'Blues',alpha = 0.3)
plt.gcf().set_size_inches(15,10)
plt.gca().axis('off')
plt.title('Slices along the z-axis')
plt.show()
```


![png](is_eda_01.png)



```python
#Create a block showing the outside surface of the imagestack
image_stack = training_inputs[:,0,:,:]
image_stack = image_stack.transpose(1,2,0)
W,H,D = image_stack.shape

xz, yz = np.meshgrid(np.linspace(0,W-1,W),np.linspace(0,H-1,H))
xz = xz.T
yz = yz.T
yx, zx = np.meshgrid(np.linspace(0,H-1,H),np.linspace(0,D-1,D))
yx = yx.T
zx = zx.T
xy, zy = np.meshgrid(np.linspace(0,W-1,W),np.linspace(0,D-1,D))
xy = xy.T
zy = zy.T

data_left = image_stack[0,:,:]/255
data_right = image_stack[-1,:,:]/255
data_back = image_stack[:,0,:]/255
data_front = image_stack[:,-1,:]/255
data_bottom = image_stack[:,:,0]/255
data_top = image_stack[:,:,-1]/255

fig = plt.figure()
ax = Axes3D(fig)
ax.plot_surface(xz,yz,D*np.ones(data_top.shape),rstride=8,cstride=8,facecolors = plt.cm.gray(data_top),shade = False)
ax.plot_surface(xz,yz,np.zeros(data_bottom.shape),rstride=8,cstride=8,facecolors = plt.cm.gray(data_bottom),shade=False)

ax.plot_surface(W*np.ones(data_right.shape),yx,zx,rstride=8,cstride=3,facecolors = plt.cm.gray(data_right),shade = False)
ax.plot_surface(np.zeros(data_left.shape),yx,zx,rstride=8,cstride=3,facecolors = plt.cm.gray(data_left),shade=False)

ax.plot_surface(xy,H*np.ones(data_front.shape),zy,rstride=8,cstride=3,facecolors = plt.cm.gray(data_front),shade = False)
ax.plot_surface(xy,np.zeros(data_back.shape),zy,rstride=8,cstride=3,facecolors = plt.cm.gray(data_back),shade=False)
ax.set_xlim3d(0, 1024)
ax.set_ylim3d(0,1024)
ax.set_zlim3d(0,1024)
plt.show()
```


![png](is_eda_02.png)



```python
#Not the smartest algorithm in the world, but it gets the job done
def Count_Convex_Shapes(image_mask):
    padded_image_mask = np.pad(image_mask,pad_width = ((1,),(1,)), mode = 'constant')
    I,J = padded_image_mask.shape
    last_pos_edges = 0
    row_shapes = 0
    col_shapes = 0
    for i in range(I):
        edge_raster = padded_image_mask[i,1:] - padded_image_mask[i,:-1]
        current_pos_edges = (edge_raster > 0).sum()
        if current_pos_edges > last_pos_edges:
            row_shapes += current_pos_edges - last_pos_edges
        last_pos_edges = current_pos_edges
    
    for j in range(J):
        edge_raster = padded_image_mask[1:,j] - padded_image_mask[:-1,j]
        current_pos_edges = (edge_raster > 0).sum()
        if current_pos_edges > last_pos_edges:
            col_shapes += current_pos_edges - last_pos_edges
        last_pos_edges = current_pos_edges
    return np.maximum(row_shapes, col_shapes)
            
```


```python
#Find the average mitochondria size
N = training_ground_truth.shape[0]
avg_area = 0
for n in range(N):
    shapes = Count_Convex_Shapes(training_ground_truth[n,0,:,:])
    area = 25*(training_ground_truth[n,0,:,:]>0).sum() #normalize to 25 nm^2 per pixel
    if shapes > 0:
        avg_area += area/shapes
avg_area /= N
print('The average mitochondrial area is', round(avg_area,1), 'square nanometers\n')
```

    The average mitochondrial area is 70013.9 square nanometers
    



```python
#Find the average mitochondria volume
def Count_Convex_Blobs(image_cube):
    N = image_cube.shape[0]
    blobs = 0
    last_shapes = 0
    for n in range(N):
        current_shapes = Count_Convex_Shapes(image_cube[n,:,:])
        if current_shapes > last_shapes:
            blobs += current_shapes - last_shapes
        last_shapes = current_shapes
    return blobs

blobs = Count_Convex_Blobs(training_ground_truth[:,0,:,:])
volume = 125*(training_ground_truth[:,0,:,:]>0).sum()
print('The average mitochondrial volume is', round(volume/blobs,1),'cubic nanometers\n')
```

    The average mitochondrial volume is 13430224.3 cubic nanometers
    



```python
#Make a histogram of the input values
plt.hist(np.ravel(training_inputs), bins = 30)
plt.show()
```


![png](is_eda_03.png)



```python
#Create two separate sets, one of data points of mitochondria, one of non-mitochondria
mitochondria_image = training_inputs[training_ground_truth >0]
nonmitochondria_image = training_inputs[training_ground_truth == 0]
```


```python
#Plot normalized histograms to see if we can just select darker regions and call them mitochondria
#Thresholding won't work, nice to know
plt.hist(np.ravel(mitochondria_image), bins = 30,alpha = 0.5, color = 'red',density = True)
plt.hist(np.ravel(nonmitochondria_image), bins = 30,alpha = 0.5, color = 'blue',density = True)
plt.show()
```
![png](is_eda_04.png)

```python
#Set up some data to get a 3D look at mitochondria
image_cube = training_ground_truth[:,0,:,:]
image_cube = image_cube.transpose(1,2,0)
image_edges = image_cube[1:,1:,1:] - image_cube[:-1,1:,1:]
image_edges += image_cube[1:,1:,1:] - image_cube[1:,:-1,1:]
image_edges += image_cube[1:,1:,1:] - image_cube[1:,1:,:-1]
x, y, z = np.where(image_edges != 0)
```


```python
%matplotlib notebook
fig = plt.figure()
ax = Axes3D(fig)
ax.plot3D(x[::20],y[::20],z[::20],alpha = 0.01, linewidth = 0,marker='.')
ax.set_xlim3d(0, 1024)
ax.set_ylim3d(0,1024)
ax.set_zlim3d(0,1024)
plt.show()
```
![png](is_eda_05.png)


