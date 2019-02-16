---
layout: post
title:  "Draw QR Code On A Background"
date:   2019-02-16 23:55:30 +0800
categories: js
---

I have a QR code, but wounded, cannot be seen clearly. I want to put the wounded QR code as a background, and draw a new QR code on top of the background.  

I also want to do some test to JavaScript canvas drawing. So I created this blog with interactive drawing functionality. In order to test pure JavaScript, no library is used.

Instructions: 
1. Set rows, columns and grid size of the QR code drawing area.
2. Browse a image file on local disk, it will show as a background and fit into to canvas with set sizes.
3. Use left mouse button to draw dark points and right mouse button to clear dark points.
4. Use Hide/Show button to hide or show background after background selected, respectively, also for further scanning.
5. Below the drawing area, a textarea is shown to represent the two dimensional data.
6. Click Set Data button to apply the data in the textarea to the drawing area if sizes match.
  

<br />
<form action='#' onsubmit="return false;">
    Rows: <input type='text' id='rowTF' value='40' style="width:50px" />
    Cols: <input type='text' id='colTF' value='40' style="width:50px" />
    Grid size: <input type='text' id='gridTF' value='15' style="width:50px" />
</form>
<form action='#' onsubmit="return false;">
    <input type='file' id='imgfile' />
    <input type='button' id='showBtn' value='Hide' style="display:none" onclick='showImage();' />
</form>
<canvas id="bg" style="position: absolute; z-index: 0;"></canvas>
<canvas id="canvas" style="position: absolute; z-index: 1; cursor: none; opacity:0.6;"></canvas>
<div id="empty"></div>
<textarea id="output" style="white-space: pre; display:none"></textarea>
<input type='button' id='dataBtn' value='Set Data' style="display:none" onclick='setData();' />

<script>
var ctx;
var datas = [];
var ROWS;
var COLS;
var GRID;
input = document.getElementById('imgfile');
input.onchange = function () {
    // fire the upload here
    loadImage();
    var showBtn = document.getElementById("showBtn");
    showBtn.style.display = "";
    var output = document.getElementById("output");
    output.style.display = "";
    var dataBtn = document.getElementById("dataBtn");
    dataBtn.style.display = "";
    init();
    draw();
};

function init() {
    var rowTF = document.getElementById('rowTF');
    var colTF = document.getElementById('colTF');
    var gridTF = document.getElementById('gridTF');
    ROWS = Number(rowTF.value);
    COLS = Number(colTF.value);
    GRID = Number(gridTF.value);
    for (var i = 0; i < ROWS; i++) {
        datas[i] = [];
        for (var j = 0; j < COLS; j++) {
            datas[i][j] = 0;
        }
    }
    outputDatas();
}
function setData() {
    var output = document.getElementById("output");
    var s = output.value;
    var cols = s.split("\n");
    if(cols.length != COLS) {
        alert("Cols not match");
        return;
    }
    var newdatas = [];
    for (var i = 0; i < ROWS; i++) {
        newdatas[i] = [];
        for (var j = 0; j < COLS; j++) {
            if(cols[i].length != ROWS) {
                alert("Rows not match");
                return;
            }
            newdatas[i][j] = Number(cols[j].substring(i, i + 1));
        }
    }
    for (var i = 0; i < ROWS; i++) {
        for (var j = 0; j < COLS; j++) {
            datas[i][j] = newdatas[i][j];
            if (datas[i][j] == 1) {
                ctx.fillStyle = "rgba(0, 0, 0, 1)";
                ctx.fillRect(i * GRID, j * GRID, GRID, GRID);
            } else {
                ctx.clearRect(i * GRID, j * GRID, GRID, GRID);
            }
        }
    }
}
function draw() {
    var empty = document.getElementById('empty');
    empty.style.width = ROWS * GRID + "px";
    empty.style.height = COLS * GRID + "px";
    var canvas = document.getElementById('canvas');
    canvas.height = ROWS * GRID;
    canvas.width = COLS * GRID;
    canvas.oncontextmenu = () => false;
    canvas.addEventListener("mousemove", move);
    canvas.addEventListener("mousedown", down);
    canvas.addEventListener("mouseup", up);
    if (canvas.getContext) {
        ctx = canvas.getContext('2d');
        ctx.strokeStyle = "none";
        ctx.strokeRect(0, 0, ROWS * GRID, COLS * GRID);
        // ctx.clearRect(45, 45, 60, 60);
        // ctx.strokeRect(50, 50, 50, 50);
    }
}
var lastRow = -1;
var lastCol = -1;
function move(e) {
    var row = Math.floor(e.offsetX / GRID);
    var col = Math.floor(e.offsetY / GRID);
    // console.log(row, col);
    if (row >= ROWS || col >= ROWS) {
        return;
    }
    if (lastRow != -1 && lastCol != -1 && (lastRow == row && lastCol == col) == false) {
        var lastGrid = datas[lastRow][lastCol];
        if (lastGrid == 1) {
            ctx.fillStyle = "rgba(0, 0, 0, 1)";
            ctx.fillRect(lastRow * GRID, lastCol * GRID, GRID, GRID);
        } else {
            ctx.clearRect(lastRow * GRID, lastCol * GRID, GRID, GRID);
        }
    }
    if (lastRow != row || lastCol != col) {
        if (isDown == 1) {
            datas[row][col] = 1;
            outputDatas();
        } else if (isDown == 2) {
            datas[row][col] = 0;
            outputDatas();
        }
        ctx.fillStyle = 'blue';
        ctx.fillRect(row * GRID, col * GRID, GRID, GRID);
        lastRow = row;
        lastCol = col;
    }
    ctx.strokeRect(0, 0, ROWS * GRID, COLS * GRID);
}
var isDown = 0;
function down(e) {
    var row = Math.floor(e.offsetX / GRID);
    var col = Math.floor(e.offsetY / GRID);
    var left, right;
    left = 0;
    right = 2;
    console.log(row, col, e.button);

    isDown = e.button === left ? 1 : 2;
    if (row >= ROWS || col >= ROWS) {
        return;
    }
    if (isDown == 1) {
        datas[row][col] = 1;
    } else if (isDown == 2) {
        datas[row][col] = 0;
    }
    outputDatas();
}
function up(e) {
    var row = Math.floor(e.offsetX / GRID);
    var col = Math.floor(e.offsetY / GRID);
    console.log(row, col);
    isDown = 0;
}
function outputDatas() {
    var output = document.getElementById("output");
    var s = "";
    for (var i = 0; i < ROWS; i++) {
        for (var j = 0; j < COLS; j++) {
            s += datas[j][i];
        }
        if(i < ROWS - 1) {
            s += "\n";
        }
    }
    output.value = s;
    if (output.scrollHeight > output.clientHeight) {
        output.style.height = output.scrollHeight + 'px';
    }
    if (output.scrollWidth > output.clientWidth) {
        output.style.width = output.scrollWidth + 'px';
    }
}

function loadImage() {
    var input, file, fr, img;

    if (typeof window.FileReader !== 'function') {
        return;
    }

    input = document.getElementById('imgfile');
    file = input.files[0];
    fr = new FileReader();
    fr.onload = createImage;
    fr.readAsDataURL(file);

    function createImage() {
        img = new Image();
        img.onload = imageLoaded;
        img.src = fr.result;
        img.width = ROWS * GRID;
        img.height = COLS * GRID;
    }

    function imageLoaded() {
        var canvas = document.getElementById("bg")
        canvas.width = ROWS * GRID;
        canvas.height = COLS * GRID;
        var ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        // alert(canvas.toDataURL("image/png"));
    }
}

function showImage() {
    var showBtn = document.getElementById("showBtn");
    var canvas = document.getElementById("bg");
    if (showBtn.value == "Hide") {
        canvas.style.display = "none";
        showBtn.value = "Show";
    } else {
        canvas.style.display = "";
        showBtn.value = "Hide";
    }
}
</script>