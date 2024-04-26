2024-04-26-Computer Vision notes

#I.The first question is as following:
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

##1.My codes：

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

##2.The solution I use

A)Prepossessing an image, which includes gray-scaling the image, blurring the image with guass-filtering, detecting the edges within the image with the canny tool, and filling the holes in the image.  The image is shown after the blurring step and the filling step, for adjustment of the parameters in the blurring and edge-detecting steps.
B)Analyzing the connected regions, including areas and boundaries thereof.
C)Identifying the region with the biggest area.
D)Showing the bounding-box of that region on the original image.


#II.The second question is:
You are provided with a video of a pantograph which provides the electrical power 
to a train (Panto2024.mp4). The electrical cable slides across the carbon brush of 
the pantograph in a zig zag manner. Your task is to graph the position of the power 
cable on the pantograph over time from the video. Note that both the suspension 
cable and the power cable are visible and that we only want to track the power 
cable. Note further that at some times in the video there are two or more power 
cables visible. We only want to track the lowest thick conductor cable which is 
powering the train. See if you can work out a way to separate the cables. Explain 
your method and any difficulties.

##1.My codes：
video = VideoReader("Panto2024.mp4");
outputVideo = VideoWriter('output_video.mp4', 'MPEG-4');
open(outputVideo);
while hasFrame(video)
frame = readFrame(video);
% Pre-processing
gray_frame = rgb2gray(frame);
binary_frame = gray_frame > 83;
edge_img = edge(binary_frame, 'canny');
% Hough
[H, theta, rho] = hough(edge_img);
P = houghpeaks(H, 10, 'threshold', ceil(0.2 * max(H(:))));
lines = houghlines(edge_img, theta, rho, P, 'FillGap', 20, 'MinLength', 100);
% select straight lines
selected_line = [];
max_left_x = Inf; % mark the leftest
for k = 1:size(lines, 2)
if ((abs(lines(k).theta) >= 2 && abs(lines(k).theta) <= 20) || (abs(lines(k).theta) >= 160 &&
abs(lines(k).theta) <= 175))
% whether endpoints on top 4/9
y_top = size(frame, 1) / 9 * 4;
if lines(k).point1(2) < y_top && lines(k).point2(2) < y_top
% check left position
if lines(k).point1(1) < 0.54 * size(frame, 2)
% update the lefest line
if lines(k).point1(1) < max_left_x
max_left_x = lines(k).point1(1);
selected_line = lines(k);
end
end
end
end
end
% draw the line on current frame
if ~isempty(selected_line)
xy = [selected_line.point1; selected_line.point2];
frame = insertShape(frame, 'Line', xy, 'LineWidth', 2, 'Color', 'red');
frame = insertMarker(frame, xy, 'x', 'Color', 'yellow', 'Size', 10);
end
% write frame into video
writeVideo(outputVideo, frame);
end
close(outputVideo);

##2. The solution I use
A) Reading the video;
B) Pre-processing each frame in the video, detecting the straight lines using Hough
transformation, including gray-scaling, making it a binary picture and detecting the edges. C) Selecting lines with angle from 2 to 20 (the power cable is appropriately vertical line, but
are not perfectly vertical);
D) Selecting lines on the top 4/9 of the frame (to filter out the lines of pantograph which are
on the bottom half). E) Selecting lines on the left 54% part of the frame (to filter out the suspension cable)
F) Selecting the leftest line (to filter out the unwanted power cable);
G) Drawing the line back to the original frame; and
H) Outputting the video (the selected straight line is the cable, the marked bottom end of the
line is the intersection of the cable and pantograph),

##3. The results
difficulties occur when a) sometimes the cable cannot be well
detected; b) the two cables meet each other or about to meet. For situation a), I think the problem is that the cable is not a perfectly straight line, so that
sometimes it may not be detected with Hough transformation. I attempted to adjust the
parameters, but I noticed that each adjustment resulted in detecting the cable in most frames, but
not all of them. For situation b), I have not figured out a general solution.

#III. The third question is as following:
Use the standard line Hough Transform to find the white line near the edge of the 
road and Mr Bean’s broomstick and back project these onto the image. Then use 
the circular Hough transform to find the centres of the 2 wheels on the Mini and 
project them back into the image. Hough transform and circular Hough transform 
functions are available in Matlab. The image is on Blackboard as MrBean2024.jpg.

#I. Detecting the white line near the edge of the road and Mr Bean’s broomstick and back
projecting these onto the image. 

##1. My codes：
% read image
img = imread('MrBean2024.jpg');
% select color
imshow(img);
title('Select the color you want to filter');
[x, y] = ginput(1); %
selected_color = impixel(img, round(x), round(y));
% define tolerance
tolerance = 30;
% calculate difference in each channel
diff_red = abs(double(img(:,:,1)) - double(selected_color(1)));
diff_green = abs(double(img(:,:,2)) - double(selected_color(2)));
diff_blue = abs(double(img(:,:,3)) - double(selected_color(3)));
% pixels with allowable tolerance in each channel
color_mask = (diff_red <= tolerance) & (diff_green <= tolerance) & (diff_blue <= tolerance);
% mark the areas with the selected color
filtered_img = img;
filtered_img(repmat(~color_mask, [1, 1, 3])) = 0;
% show result of filtering
figure;
subplot(1,4,1), imshow(img), title('Original Image');
subplot(1,4,2), imshow(filtered_img), title('Filtered Image');
% gray, canny, Hough
gray_img = rgb2gray(filtered_img);
BW = edge(gray_img, 'canny');
subplot(1,4,3),imshow(BW), title('edge Image');
[H,theta,rho] = hough(BW);
% find peaks
P = houghpeaks(H, 50, 'Threshold', 0.3*max(H(:)));
% locate straight lines
lines = houghlines(BW,theta,rho,P,'FillGap',5,'MinLength',50);
% mark staright lines on original image
subplot(1,4,4),imshow(img), title('marked Image 1'), hold on
for k = 1:length(lines)
xy = [lines(k).point1; lines(k).point2];
plot(xy(:,1), xy(:,2), 'LineWidth', 2, 'Color', 'red');
end

##2. The solution I use
It can be seen that the the white line near the edge of the road and Mr Bean’s broomstick
share similar color, and also they are shaped as straight lines. I select the color on the stick, filter
the image on the basis of the selected color, pre-process the filtered image, use Hough transform
to detect the straight lines, and mark the detected lines on the original image. 

##3. Results
The threshold of the length should not be too long or too short; if set too long, it may filter
out the white line on the road, and if set too short, unwanted lines may remain.
