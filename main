clf; figure('Color','w','Units','normalized','Position',[0.18 0.14 0.6 0.72]);
ax = axes('Position',[0 0 1 1]); hold(ax,'on'); axis(ax,'equal','off'); view(ax, -35, 28);
set(gcf,'Renderer','opengl');

%% -------------------- STYLE --------------------
rows  = 4;                 % tiles vertically
cols  = 4;                 % tiles horizontally

L     = 10;                % SIDE of square board (local units)
W = L; H = L;              % keep panel square
T     = 0.1;               % physical thickness along -Z (extrusion)

% Colors (light theme)
colFrame   = [0.99 0.99 0.90];
colEdge    = [0.55 0.58 0.62];
colBezel   = [0.94 0.97 0.99];
tileC1     = [0.10 0.78 0.92];
tileC2     = [0.04 0.88 0.78];
tileAlpha  = 0.60;

% Borders & spacing
outerMargin = 0.18;        % bezel inset
gap         = 0.014;       % tiles close but not touching
tileInset   = 0.10;        % inset within a cell
lineW       = 0.7;         % edge width

% Optional in-plane rotation to match the 2D look
globalRotDeg  = -15;

%% -------------------- GEOMETRY HELPERS --------------------
rotM3z = @(deg) [cosd(deg) -sind(deg) 0; sind(deg) cosd(deg) 0; 0 0 1];
applyR = @(P) (P*rotM3z(globalRotDeg).');

P = W/2; Q = H/2;

% Front and back rectangles (z=0 front, back at z=-T)
panelF = [ -P -Q 0;  P -Q 0;  P  Q 0;  -P  Q 0 ];
panelB = [ -P -Q -T; P -Q -T; P  Q -T; -P  Q -T ];

% Apply in-plane rotation
panelF = applyR(panelF); panelB = applyR(panelB);

% Bezel polygon on the front face
bezelF = [ -(P-outerMargin) -(Q-outerMargin) 0;
            +(P-outerMargin) -(Q-outerMargin) 0;
            +(P-outerMargin) +(Q-outerMargin) 0;
            -(P-outerMargin) +(Q-outerMargin) 0 ];
bezelF = applyR(bezelF);

%% -------------------- DRAW BOARD --------------------
% Back face
patch('Faces',[1 2 3 4],'Vertices',panelB, ...
      'FaceColor',colFrame*0.96,'EdgeColor',colEdge,'LineWidth',lineW);

% Sides (right and top are naturally visible from top-right view)
% Order corners: [BL BR TR TL] in panelF
BL = panelF(1,:); BR = panelF(2,:); TR = panelF(3,:); TL = panelF(4,:);
BLb= panelB(1,:); BRb= panelB(2,:); TRb= panelB(3,:); TLb= panelB(4,:);

% Right side (BR→TR)
Vright = [BR; TR; TRb; BRb];
patch('XData',Vright(:,1),'YData',Vright(:,2),'ZData',Vright(:,3), ...
      'FaceColor',colFrame*0.90,'EdgeColor',colEdge,'LineWidth',lineW);

% Top side (TR→TL)
Vtop = [TR; TL; TLb; TRb];
patch('XData',Vtop(:,1),'YData',Vtop(:,2),'ZData',Vtop(:,3), ...
      'FaceColor',colFrame*0.87,'EdgeColor',colEdge,'LineWidth',lineW);

% Left and bottom sides (draw but they'll be less visible)
Vleft  = [TL; BL; BLb; TLb];
Vbot   = [BL; BR; BRb; BLb];
patch('XData',Vleft(:,1),'YData',Vleft(:,2),'ZData',Vleft(:,3), ...
      'FaceColor',colFrame*0.93,'EdgeColor',colEdge,'LineWidth',lineW);
patch('XData',Vbot(:,1),'YData',Vbot(:,2),'ZData',Vbot(:,3), ...
      'FaceColor',colFrame*0.93,'EdgeColor',colEdge,'LineWidth',lineW);

% Front face
patch('Faces',[1 2 3 4],'Vertices',panelF, ...
      'FaceColor',colFrame,'EdgeColor',colEdge,'LineWidth',lineW);

% Slim bezel on top of front face (slightly lifted to avoid z-fighting)
bz = bezelF; bz(:,3) = bz(:,3) + 1e-3;
patch('Faces',[1 2 3 4],'Vertices',bz, ...
      'FaceColor',colBezel,'EdgeColor',colEdge*0.85,'LineWidth',lineW);

%% -------------------- TILE GRID (aligned squares, diagonal two-tone) --------------------
% Grid in bezel XY (compute in unrotated coords, then rotate & place on z=0)
bezel0 = [ -(P-outerMargin) -(Q-outerMargin);
            +(P-outerMargin) -(Q-outerMargin);
            +(P-outerMargin) +(Q-outerMargin);
            -(P-outerMargin) +(Q-outerMargin) ];

Bx = linspace(bezel0(1,1), bezel0(2,1), cols+1);
By = linspace(bezel0(1,2), bezel0(4,2), rows+1);

cellCx = @(i) (Bx(i)+Bx(i+1))/2;
cellCy = @(j) (By(j)+By(j+1))/2;
cellW  = @(i) (Bx(i+1)-Bx(i)) - gap;
cellH  = @(j) (By(j+1)-By(j)) - gap;

for j = 1:rows
    for i = 1:cols
        w = max(cellW(i),0) * (1 - tileInset);
        h = max(cellH(j),0) * (1 - tileInset);
        cx = cellCx(i);  cy = cellCy(j);

        % Local square (BL, BR, TR, TL) at z=0
        sq0 = [ cx - w/2, cy - h/2, 0;   % BL (1)
                cx + w/2, cy - h/2, 0;   % BR (2)
                cx + w/2, cy + h/2, 0;   % TR (3)
                cx - w/2, cy + h/2, 0 ]; % TL (4)

        % Rotate into board orientation and slightly lift above bezel
        sq = applyR(sq0); sq(:,3) = sq(:,3) + 2e-3;

        % Triangles covering the square
        tri1 = sq([1 2 3],:);  % lower-right
        tri2 = sq([1 3 4],:);  % upper-left

        patch('XData',tri1(:,1),'YData',tri1(:,2),'ZData',tri1(:,3), ...
              'FaceColor',tileC1,'FaceAlpha',tileAlpha,'EdgeColor','none');
        patch('XData',tri2(:,1),'YData',tri2(:,2),'ZData',tri2(:,3), ...
              'FaceColor',tileC2,'FaceAlpha',tileAlpha,'EdgeColor','none');

        % Diagonal line from TOP-RIGHT to BOTTOM-LEFT (TR -> BL)
        pts = sq([3 1],:);
        plot3(pts(:,1),pts(:,2),pts(:,3), ...
              'Color',colEdge*0.7,'LineWidth',0.7);

        % Thin square outline
        patch('XData',sq([1 2 3 4],1),'YData',sq([1 2 3 4],2),'ZData',sq([1 2 3 4],3), ...
              'FaceColor','none','EdgeColor',colEdge*0.6,'LineWidth',0.6);
    end
end

%% -------------------- LIGHTING & CAMERA --------------------
camlight headlight; material([0.4 0.8 0.3]);
axis vis3d; daspect([1 1 0.3]); % slight vertical compression for depth feel

% Fit view / padding
[allX, allY, allZ] = deal([]);
for k = 1:numel(ax.Children)
    if isprop(ax.Children(k),'XData'), allX = [allX; ax.Children(k).XData(:)]; end %#ok<AGROW>
    if isprop(ax.Children(k),'YData'), allY = [allY; ax.Children(k).YData(:)]; end %#ok<AGROW>
    if isprop(ax.Children(k),'ZData'), allZ = [allZ; ax.Children(k).ZData(:)]; end %#ok<AGROW>
end
pad = 0.07*max(range([allX allY],1));
xlim([min(allX)-pad, max(allX)+pad]);
ylim([min(allY)-pad, max(allY)+pad]);
zlim([min(allZ)-T*0.2, max(allZ)+T*0.8]);
grid off;
