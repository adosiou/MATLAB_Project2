%%% 2nd exercise, Geomodelopoihsh %%%
%%% Anna Dosiou, 222305 %%%
%% 1st: read the columns of humid.txt %%
clear all;
close all;
clc;
load humid.txt;
% choose columns %
lat=humid(:,2);
lon=humid(:,3);
relat_humid=humid(:,9);

%% 2nd: plots for humidity with three different methods of interpolation %%
load aktogr.txt;
aktogr = aktogr(:, 1:2);
xi = linspace(min(lon), max(lon), 100);
yi = linspace(min(lat), max(lat), 100);
% convert the data to table %
[X,Y] = meshgrid(xi,yi);

% nearest neighbor interpolation method %
Inter_nearest = griddata(lon, lat, relat_humid, X, Y, 'nearest');
figure;
hold on;
contourf(X, Y, Inter_nearest);
plot(aktogr(:, 1), aktogr(:, 2), 'k');
title('Contour Plot - Nearest Interpolation');
colormap(jet);
colorbar;
hold off;

% linear interpolation method %
Inter_linear = griddata(lon, lat, relat_humid, X, Y, 'linear');
figure;
hold on;
contourf(X, Y, Inter_linear);
plot(aktogr(:, 1), aktogr(:, 2), 'k');
title('Contour Plot - Linear Interpolation');
colormap(jet);
colorbar;
hold off;

% cubic interpolation method %
Inter_cubic = griddata(lon, lat, relat_humid, X, Y, 'cubic');
figure;
hold on;
contourf(X, Y, Inter_cubic);
plot(aktogr(:, 1), aktogr(:, 2), 'k');
title('Contour Plot - Cubic Interpolation');
colormap(jet);
colorbar;
hold off;

%% 3rd: read the tiff of Greece and the legend of corine land cover %%
f=importdata('clc_legend.xls');

x1=f.data(:,1);  % first column, times of land use %
x2=f.data(1:48,10:12);  % columns J,K,L,  RGB color triplets (0-1) %
x3=f.data(1:48,:); % all columns for rows 1-49 %
x4=f.textdata(3:49,5);  % land use types names %
tf = Tiff('greece.tif','r');
D = tf.read;

%% 4th: frequency and percent of land use types %%
Ny=numel(D(:,1)); 
Nx=numel(D(1,:));

k=0;
for i = 1:Ny
    for j = 1:Nx
        if D(i,j) == 3;
            k = k+1;
        end
    end
end

t = zeros(46,1);
for i = 1:Ny
    for j = 1:Nx
        for g = 1:48
            if D(i,j) == g && g ~= 1 
                t(g) = t(g)+1;
            end
        end
    end
end
frequencies = t;
percentages = (t / sum(t)) * 100;

% bar plot percent-land use type%
figure; 
bar(percentages);
title('Land Use Types','Color', 'r','FontSize',10);
xlabel('Land Use Categories');
ylabel('Percentage');
ax = gca; % current axes
ax.FontSize = 7; 
ax.TickDir = 'out';
ax.TickLength = [0.01 0.02];
ax.XLim = [0 50]; 
ax.XTick = 0:1:50;
ax.XTickLabelRotation = 90;
ax.XTickLabel = x4;

colormap(ax, 'hot');
c = colorbar;
c.Ticks = [0 1];
c.TickLabels = {'Exclude', '100%'};

%% 5th: zoom to Evoia and to the land use types 15-25 %%
zoom_Evoia = D(1570:2000, 1290:1920);

evoia_15_25 = zoom_Evoia;
evoia_15_25(evoia_15_25 < 15 | evoia_15_25 > 25) = NaN;
h = figure;
im = image(evoia_15_25);
xlabel('longitude');
ylabel('latitude');
set(gca,'XTickLabel',[],'YTickLabel',[])
title('Evoia Land Use Types 15-25');
set(gcf,'ColorMap',x2);
p = colorbar('Location','EastOutside','YTick',15:1:25,'YTickLabel',f.textdata(17:1:27,5),'fontsize',5);
%set(h, 'Position',[0 0 1000 500]);
set(p,'ylim',[14 25])
print(h,'-dtiff','evoia_corine_zoom')
size(zoom_Evoia);

%% 6th: the 3 predominant types in percentages of occurrence in Evoia

Ny=numel(zoom_Evoia(:,1)); 
Nx=numel(zoom_Evoia(1,:));

k=0;
for i = 1:Ny
    for j = 1:Nx
        if zoom_Evoia(i,j) == 3;
            k = k+1;
        end
    end
end

t = zeros(46,1);
t_s = cell(46, 1);
t_s(:) = {''};

for i = 1:Ny
    for j = 1:Nx
        for g = 1:48
            if zoom_Evoia(i,j) == g && g ~= 1; 
                t(g) = t(g)+1;
                value_from_x4 = x4(x1 == zoom_Evoia(i,j));
                t_s(g) = value_from_x4(1);
            end
        end
    end
end

percentages_evoia = (t / sum(t)) * 100;
[sorted_percentages, sorting_indices] = sort(percentages_evoia, 'descend'); % descending order of percentages
sorted_t_s = t_s(sorting_indices); % sorting of land use types by percentages
top_t_s = sorted_t_s(1:3);  % the 3 predominant land use types for highest percentages

top_percentages = sorted_percentages(1:3);  % the 3 highest percentages
disp('The 3 predominant types of occurrence in Evoia are: ');
disp(top_t_s);  % display the the 3 predominant land use types
disp('and their percentages are: ');
disp(top_percentages); % display the percentages of the 3 predominant land use types

%% 7th: Creation of a table and save as csv
codes_landUse = f.textdata(18:28,2); % codes of land use types %
names_landUse = f.textdata(18:28,5);  % land use types names %

final_question_array = [codes_landUse, names_landUse, num2cell(percentages_evoia(15:25))];

sorted_final_question_array = sortrows(final_question_array, -3);
T = cell2table(sorted_final_question_array, 'VariableNames', {'Code', 'Name', 'Percentage'});
writetable(T, 'sorted_array.csv');
