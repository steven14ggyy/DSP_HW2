# 翁啟文 <span style="color:red">(103061112)</span>

# HW2 / Image Filtering and Corner Detection

## Overview
The project is related to corner detection. By digital signal processing, corners in an image can be found out and labeled as red circles on the image. To do so, we use gradient filters and a Gaussian filter, calculate "corner response function" (R), transform R to a binary image,  and finally find local maximums, where corners appear.


## Implementation
1. MyHarrisCornerDetector.m  
Based on Harris' method, first we have to find image gradients in two directions (x and y). While we can use for loop to calculate the differences between each point and its neighbors, conducting convolutions of an input image and gradient filters may be a better choice. Before convolutions, transform RGB image into gray scale image. Use build-in matlab function imfilter() to get Ix and Iy. Note that Ix and Iy represent gradients of the whole input image, which means they combine the gradients of all pixels on the input image, so they are matrix, whose sizes are same as the input image. 
```Matlab
% get image gradient
% [Your Code here] 
%Transform into gray scale image
I = I(:,:,1)*0.299+I(:,:,2)*0.587+I(:,:,3)*0.114;
% calculate Ix
Ix = imfilter(I, dx);
% calcualte Iy
Iy = imfilter(I, dy);
```
Next, based on Taylor expansion of SSD(x,y), the weighted __sum of squared differences__, we also need to find out Ix2, Iy2 and Ixy, which are second derivatives based on x and y. Use element-by-element multiplications to get Ix2, Iy2 and Ixy, which are components of second moment matrix. Pass a Gaussian filter to smooth them.
```Matlab
% calculate Ix2  
Ix2 = Ix.*Ix;
Ix2 = imfilter(Ix2, g);
% calculate Iy2
Iy2 =  Iy.*Iy;
Iy2 = imfilter(Iy2, g);
% calculate Ixy
Ixy =  Ix.*Iy;
Ixy = imfilter(Ixy, g);
```
We plot Ixy on the screen and demo to TAs:  
<img src= https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/corner_Ixy.jpg width="40%"/>

Next, calculate corner response function R = det(M)-alpha*trace(M)^2. __Note that the determinant of M is (Ix2.*Iy2 - Ixy.*Ixy) and trace(M) is (Ix2.*Iy2).__ And we scale R to 0~1000. Use ordfilt2() to find a maximum value in each separate domain (in our case, the size of domain is sze * sze = 13 * 13). There is a simple diagram showing how to find local maximums by comparison:  
<img src= https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/results/explain_1.png width="70%"/>  
After comparison, store value "1" at the same (row, col) in RBinary array to represent positions of local maximums.  
```Matlab
% calculate R
R = Ix2.*Iy2-Ixy.*Ixy - alpha*(Ix2+Iy2).^2;

%% make max R value to be 1000
R=(1000/max(max(R)))*R; % be aware of if max(R) is 0 or not

%% using B = ordfilt2(A,order,domain) to complement a maxfilter
sze = 2*r+1; % domain width 
% calculate MX
MX = ordfilt2(R,sze^2,ones(sze,sze));

% find local maximum.
% calculate RBinary
RBinary = zeros(xmax,ymax);
for i = xmin+1:xmax-1
    for j =  ymin+1:ymax-1
        if(R(i,j)>Thrshold && R(i,j)==MX(i,j))
            RBinary(i,j) = 1;
        end
    end
    a=1;
end
```
Then, we exclude the local maximum points which exist along image's edges, and record (row, col) of the rest. Finaly, we display the result and mark corner points by circle signs.
```Matlab
%% get location of corner points not along image's edges
offe = r-1;
count=sum(sum(RBinary(offe:size(RBinary,1)-offe,offe:size(RBinary,2)-offe))); % How many interest points, avoid the image's edge   
R=R*0;
R(offe:size(RBinary,1)-offe,offe:size(RBinary,2)-offe)=RBinary(offe:size(RBinary,1)-offe,offe:size(RBinary,2)-offe);
[r1,c1] = find(R);
  
%% Display
figure(3)
imagesc(uint8(frame));
hold on;
plot(c1,r1,'or');
```

### Results:    
|Input|Output|  
|---------------|---------------|   
|<img src= https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/Im.jpg width ="100%"/>|<img src=https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/Im_corner.png width="100%"/>|   
|<img src= https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/chess.png width="100%"/>|<img src=https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/chess_corner.png width="100%"/>|  
|<img src= https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/DSC_0116_1.JPG width="100%"/>|<img src=https://github.com/steven14ggyy/DSP_Lab_HW2/blob/master/data/DSC_0116_1_corner.JPG width="100%"/>|
