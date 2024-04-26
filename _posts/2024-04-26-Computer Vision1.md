2024-04-26-Computer Vision notes

The second question is:
You are provided with a video of a pantograph which provides the electrical power 
to a train (Panto2024.mp4). The electrical cable slides across the carbon brush of 
the pantograph in a zig zag manner. Your task is to graph the position of the power 
cable on the pantograph over time from the video. Note that both the suspension 
cable and the power cable are visible and that we only want to track the power 
cable. Note further that at some times in the video there are two or more power 
cables visible. We only want to track the lowest thick conductor cable which is 
powering the train. See if you can work out a way to separate the cables. Explain 
your method and any difficulties.

My codes：
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

2. The solution I use
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

3. The results
difficulties occur when a) sometimes the cable cannot be well
detected; b) the two cables meet each other or about to meet. For situation a), I think the problem is that the cable is not a perfectly straight line, so that
sometimes it may not be detected with Hough transformation. I attempted to adjust the
parameters, but I noticed that each adjustment resulted in detecting the cable in most frames, but
not all of them. For situation b), I have not figured out a general solution.

