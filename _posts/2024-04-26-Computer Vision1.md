#2024-04-26-Computer Vision notes

#The question is as following:
Many modern vehicles can detect and recognize street signs to advise on speed 
limits and other road information. Your mission is to automatically locate the street 
signs in the following image (see StreetSigns2024.zip from Blackboard). You may 
try a 2D cross-correlation with a template matching the sign border, but you will 
probably need to ‘chamfer’ the template by convolving with a Gaussian or some 
other blurring function. You’ll also need a good edge detector such as the Canny 
Edge detector. Test your method on the other example sign images from 
numberplates2022.zip on Blackboard and show the results. 
In your report discuss methods used, problems encountered, performance, and 
possible solutions. Comment on the problems encountered in sign extraction and 
the difficulties in designing a general sign detector.
![Street Sign](images/Street sign.jpg)

#1.My codes：

% read imgage
img = imread('image13.jpg');

% pre-process
img_gray = rgb2gray(img);
blurred_img = imgaussfilt(img_gray, 0.5); 
figure, subplot(1,3,1), imshow(blurred_img);
img_edge = edge(blurred_img, 'canny', [0.1 0.3]);
img_filled = imfill(img_edge, 'holes'); 
subplot(1,3,2), imshow(img_filled)

% analyzing connected regions
cc = bwconncomp(img_filled); 
stats = regionprops(cc, 'Area', 'BoundingBox', 'FilledImage');

% find region with biggest area
areas = [stats.Area];
[max_area, max_idx] = max(areas);

% show result
subplot(1,3,3), imshow(img);
hold on;
rectangle('Position', stats(max_idx).BoundingBox, 'EdgeColor', 'r', 'LineWidth', 2); 
hold off;

#2.The solution I use

A)Prepossessing an image, which includes gray-scaling the image, blurring the image with guass-filtering, detecting the edges within the image with the canny tool, and filling the holes in the image.  The image is shown after the blurring step and the filling step, for adjustment of the parameters in the blurring and edge-detecting steps.
B)Analyzing the connected regions, including areas and boundaries thereof.
C)Identifying the region with the biggest area.
D)Showing the bounding-box of that region on the original image.

#3. The results
![Processed image](images/Image0.jpg)