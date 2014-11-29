moveableobject
==============

canvas drawing



var pageContainer, footerContainer,menuContainer, page0, dragContainer, dragActivity, ActivityName, flipDisp=[],random_obj=[],draggable_objects, setCount = 1, draggingEnabled = false;
var randomObj1,randomObj2;
var audioinst=new Audio('./audio/inst.mp3');
var eventDown = "mousedown";
var eventMove = "mousemove";
var eventUp = "mouseup";
var toggled = false;
var paintingStart = false;
var painting = false;
var eraserEnable = false;
var canvas;
var ballTopPos = [];
var ballLeftPos = [];
var objWidth, objHeight;
var ObjPos = new Array();
var isJustClicked = false;
//var colorArray=["ladybug1","ladybug2","ladybug3","ladybug4","ladybug5","ladybug6","ladybug7","ladybug8","ladybug9"];
var colorArray=["img1","img2","img3","img4","img5","img6","img7","img8","img9","img10","img11","img12","img13","img14"];
var prevColor;
$(document).ready(function(){
	/* To Inactivate TAB Key */
	audioinst.play();	
	document.onKeyDown = function(){
		if(window.event && window.event.keyCode == 9) 
		{
			return false; 
		}
	}
        pageContainer = $("<div id=\"interactive-container\" />");
        menuContainer = $("<div id=\"page-container\" />");
	footerContainer = $("<div class=\"contentHolder\"><div class=\"container\"><div class=\"contentFeedback\"><div class=\"btReset\"><div class=\"icoReset\"></div></div><div class=\"btInfo\"><div class=\"icoInfo\"></div></div><div class=\"btFeedback\"><div class=\"icoFeed\"></div></div><div class=\"score\"><div id=\"contCorrect\">0 <img src=\"images/IconCorrect.png\"></div><div id=\"contIncorrect\">0 <img src=\"images/IconIncorrect.png\"></div></div></div></div>");
        page0 = $("<div id=\"page-0\" class=\"page\" />");
	blocker = $("<div id=\"blocker\" />");
	instruction = $("<div id=\"question_blocker\" onclick=\"closeBtn()\"></div><div class=\"question\"><div style=\"position:absolute; left:1.5%; top:3%; opacity:0.6;\" id=\"instructions-button\"><img src=\"images/btInfoOn.png\" width=\"32\" height=\"32\"/></div><span style=\"text-align:left;font-size:30px; font-family:myFontfamily; left:60px; line-height:1.4em; width:645px; position:absolute; top:50px;\">Put object that are the same in one way in groups.<br/>Tap on the pencil to draw loops around the groups.<br/>Tap on the eraser to clear all the loops.<br/>To play with new object, tap on the New Set button.</span><div class=\"closeBtn\" onclick=\"closeBtn()\">&times;</div><div style=\"position:absolute;  left:46%; top:73%;\" onclick=\"AudioState()\"><img src=\"images/audioICON.png\" width=\"50px\" height=\"54px\" /></div></div>");
	blocker.css({'position':'absolute','left':'0px','top':'0px','width':'1024px','height':'768px','z-index':'1100','display':'none'})
        menuContainer.append(page0);
        pageContainer.append(menuContainer);
	pageContainer.append(footerContainer);
        $('body').append(pageContainer);
	page0.append(blocker);
	page0.append(instruction);
	draw = $("<div id=\"pen\" class=\"draw-1\" > <div class=\"button\" id=\"toggleDrawing\"> </div></div>");
	erase = $("<div id=\"eraser\" class=\"erase-1\"><div class=\"button\" id=\"clearCanvas\"></div></div>");
	newset = $("<div id=\"new\" class=\"newset-1\" > <div class=\"button\" id=\"toggleDrawing1\"><i class=\"icon-undo\"></i>&nbsp;\n </div>\n</div>");
	page0.append(draw);
	page0.append(erase);
	page0.append(newset);
	draggable_objects = $("<div id=\"draggable_objects\" />");
	draggable_boundary = $("<div id=\"draggable_boundary\"></div>");
	draggable_boundary.css({'position':'absolute','left':'0px','top':'0px','width':'1024px','height':'570px'})
	page0.append(draggable_boundary);
	canvas = $('<canvas/>').attr({width: 1024, height: 570, id: "myCanvas"});
	page0.append(draggable_objects);
	draggable_boundary.append(canvas);
	context = canvas.get(0).getContext("2d");
	Activity_Type = "start";
	$(".icoInfo").on('click',function () {
		toggleClassButton($(this),'icoInfoMove');
		$('.question,#question_blocker').show();
		$('.question,#question_blocker').css({'z-index':'8000'})
		audioinst.currentTime=0;
		audioinst.play();	    
	})
	if ('ontouchstart' in document.documentElement) {
		eventDown = 'touchstart';
	}
	if('ontouchmove' in document.documentElement) {
		eventMove = 'touchmove';
	}
	if('ontouchstart' in document.documentElement) {
		eventUp = 'touchend';
	}
	
	randomObj1 = colorArray[Math.floor(Math.random()*colorArray.length)];
	randomObj2 = colorArray[Math.floor(Math.random()*colorArray.length)];
	
	if (randomObj1 == randomObj2) {
		randomObj1 = colorArray[Math.floor(Math.random()*colorArray.length)];
		randomObj2 = colorArray[Math.floor(Math.random()*colorArray.length)];
	}
	
	console.log(randomObj1,randomObj2);
	
	$('#eraser').bind("click",function(){
		$(this).addClass('active')
		currentBtn = $(this);
		clearInterval(t1);
		var t1=setTimeout(function(){
			currentBtn.removeClass('active');
			clearInterval(t1);
		},100)
		$("#draggable_boundary").css("z-index",0);
		context.clearRect(0, 0, canvas.width(), canvas.height());
		$('#myCanvas').css({'background-image':'none'});
		if (draggingEnabled) {
			$('.icoReset').css({'opacity':'1','cursor':'pointer'});
			$('.icoReset').bind('click',resetfn);
		}
		else{
			$('.icoReset').css({'opacity':'0.5','cursor':'default'});
			$('.icoReset').unbind('click',resetfn);
		}
		if ($('#pen').hasClass('active')) {
			DragItem = false;
			paintingStart = true;
			toggled = true;
	         	$('#draggable_boundary').css({'z-index':'4000'});
			$('.icoReset').css({'opacity':'1','cursor':'pointer'});
			$('.icoReset').bind('click',resetfn);
		}		
	});
	$('#new').bind("click",newset_function);
	$('#pen').bind("click", function(){
		$(this).toggleClass('active');
		if (toggled) {
			paintingStart = false;
			painting = false;
			$('#draggable_boundary').css({'z-index':'0'});
			toggled = false;
			$('.icoReset').css({'opacity':'1','cursor':'default'});
			$('.icoReset').bind('click',resetfn);
		}
		else
		{
			DragItem = false;
			paintingStart = true;
	         	$('#draggable_boundary').css({'z-index':'4000'});
			toggled = true;
			$('.icoReset').css({'opacity':'1','cursor':'pointer'});
			$('.icoReset').bind('click',resetfn);
		}
	});
	$('#myCanvas').bind(eventDown, function(e){
		console.log("down")
		if(paintingStart) {
			isJustClicked = true;
			painting = true;
			element = $(event.target);
			parentOffset = element.parent().offset();
			if(eventDown == "mousedown") {
				lastX = e.pageX - this.offsetLeft - parentOffset.left;
				lastY = e.pageY - this.offsetTop - parentOffset.top;
			}else{
				lastX = e.originalEvent.touches[0].pageX - this.offsetLeft - parentOffset.left;
				lastY = e.originalEvent.touches[0].pageY - this.offsetTop - parentOffset.top;
			}
		}
	});
	$(window).bind(eventUp, function(e){
	    painting = false;
	    isJustClicked = false;
	});
	$(window).bind(eventMove, function(e){
	    if (painting) {
		if(eventMove == "mousemove") {
			mouseX = e.pageX - document.getElementById("myCanvas").offsetLeft - parentOffset.left;
			mouseY = e.pageY - document.getElementById("myCanvas").offsetTop - parentOffset.top;
		}
		else{
			mouseX = e.originalEvent.touches[0].pageX - document.getElementById("myCanvas").offsetLeft - parentOffset.left;
			mouseY = e.originalEvent.touches[0].pageY - document.getElementById("myCanvas").offsetTop - parentOffset.top;
		}
		// find all points between        
		var x1 = mouseX,
		    x2 = lastX,
		    y1 = mouseY,
		    y2 = lastY;
		var lineThickness = 5;
		if (x1 < lineThickness) x1 = lineThickness;
		else if (x1 >= $("#myCanvas").width()-lineThickness) x1 = $("#myCanvas").width()-lineThickness;
		if (y1 < lineThickness) y1 = lineThickness;
		else if (y1 >= $("#myCanvas").height()-lineThickness) y1 = $("#myCanvas").height()-lineThickness;
		if (isJustClicked) {
			context.beginPath();
			context.moveTo(x1, y1);
			context.lineWidth = lineThickness;
			isJustClicked = false;
		}
		else context.lineTo(x1, y1);
		context.strokeStyle = "#000";
		context.stroke();
		lastX = mouseX;
		lastY = mouseY;
	    }
	});
init();	
/*Saved state*/
eventBroker = _({}).extend(require('chaplin/lib/event_broker'));
eventBroker.publishEvent("#fetch", { type : 'state' }, function(state) {
	if (state) {
		_.each(JSON.parse(state), function(value, key, list) {
		if(key == "objArray" && value) {
			$('#draggable_objects').html(value);
			scatterRandom(true);
		}
		if (key == "origArray") {
			for(i=0;i<value.length;i++){
				random_obj[i].OrgX = value[i].x;
				random_obj[i].OrgY = value[i].y;
			}
		}
		if (key == "canvasData") {
			var canvasEle = document.getElementById('myCanvas');
			var ctx = canvasEle.getContext("2d");
			var image = new Image();
			image.src = value;
			image.onload = function() {
				ctx.drawImage(image, 0, 0);
			};
                    }
		if (key == "canvasDrawEnable") {
		     if (value) {
			$('#draggable_boundary').css({'z-index':'4000'});
			toggled = true;
			paintingStart = true;
			$('#myCanvas').css({'z-index':'4000'});
		     }
		     else{
			$('#myCanvas').css({'z-index':'1000'});
		     }
		}
		    if (key == "new") {
			if(value){$('#new').addClass('active')};
		    }
		    if (key == "pen") {
			if(value){$('#pen').addClass('active')};
		    }
		    if (key == "erase") {
			if(value){$('#eraser').addClass('active')};
		    }
		    if (key == "resetVal") {
			$(".icoReset").css({"opacity": value});
			if ($(".icoReset").css('opacity')) {
				$(".icoReset").css({'cursor':'pointer'});
				$(".icoReset").bind('click',resetfn)
			}
		    }
		    if (key == 'draggingEnabled') {
			draggingEnabled = value;
		    }
		});
	}
});
eventBroker.subscribeEvent('#doSave', function(state) {
	var state = {};
	var origArray = [];
	for(i=0;i<random_obj.length;i++){
		origArray.push({x:random_obj[i].OrgX, y:random_obj[i].OrgY});
	}
	state = {
		"canvasDrawEnable":paintingStart,
		"objArray":$('#draggable_objects').html(),
		"origArray":origArray,
		"canvasData":document.getElementById('myCanvas').toDataURL(),
		"pen":$("#pen").hasClass("active"),
		"new":$("#new").hasClass("active"),
		"erase":$("#eraser").hasClass("active"),
		"resetVal": $(".icoReset").css("opacity"),
		"draggingEnabled":draggingEnabled
		}
	/**/
	var message = {
	type : 'state',
	data : JSON.stringify(state)
	};
	eventBroker.publishEvent("#save", message);
	});

	
	
	
});
function resetfn(){
	Activity_Type = "reset";    
	toggled =false;
	DragItem = true;
	$("#draggable_boundary").css("z-index",0);
	context.clearRect(0, 0, canvas.width(), canvas.height());
	draggingEnabled = false;
	$('#pen, #eraser').removeClass('active');
	paintingStart = false;
	for (i=0; i<random_obj.length; i++){
		random_obj[i].css({'left':random_obj[i].OrgX, 'top':random_obj[i].OrgY});
	}
	$('#myCanvas').css({'background-image':'none'});
	$(".icoReset").removeClass('icoResetMove');
        var to=setTimeout(function(){
			clearInterval(to);
			$(".icoReset").addClass('icoResetMove');
			$(".icoReset").css({'opacity':'0.5', 'cursor':'default'});
			$(".icoReset").unbind('click',resetfn);
		},100)
}
function init() {
	toggled =false;
	$("#draggable_boundary").css("z-index",0);
	context.clearRect(0, 0, canvas.width(), canvas.height());
	$('#myCanvas').css({'background-image':'none'});
	paintingStart = false;
	scatterRandom(false);
}
function closeBtn(){
	$('.question,#question_blocker').hide();
	$(".icoInfo").addClass('icoInfoMove');
	try{audioinst.pause();} catch(e){}
}
function saveFunction(){
	eventBroker.publishEvent("#doSave");
}
function AudioState() {
	if (audioinst.paused == false) {
		audioinst.pause();
		audioinst.currentTime = 0;
		audioinst.play();
	}
	else{
		audioinst.play();
	}
}
function toggleClassButton(element,className){
	var currentButton=element;
	if(!currentButton.hasClass(className)){
		currentButton.addClass(className);
	}else{
		currentButton.removeClass(className);	
	}
}	
function newset_function() {
	paintingStart = false;
	DragItem = true;
	painting = false;
	$(this).addClass('active')
	currentBtn = $(this);
	//console.log(currentBtn);
	clearInterval(t);
	var t=setTimeout(function(){
		currentBtn.removeClass('active');
		clearInterval(t);
	},100)
	$('#draggable_boundary').css({'z-index':'0'});
	toggled = false;
	ObjPos = new Array();
	randomObj1=0,randomObj2=0;
	$('#pen, #eraser').removeClass('active');
	$('.icoReset').css({'opacity':'0.5','cursor':'default'});
	$('.icoReset').unbind('click',resetfn);
	context.clearRect(0, 0, canvas.width(), canvas.height());
	randomObj1 = colorArray[Math.floor(Math.random()*colorArray.length)];
	randomObj2 = colorArray[Math.floor(Math.random()*colorArray.length)];
	
	if (randomObj1 == randomObj2) {
		randomObj1 = colorArray[Math.floor(Math.random()*colorArray.length)];
		randomObj2 = colorArray[Math.floor(Math.random()*colorArray.length)];
	}
	init();
}
function scatterRandom(isObjAlready){
	{
	    obj_no = 10;//Minimum;
	    left_pos = 15;
	    top_pos = draggable_boundary.position()
	    width_pos = 904;
	    height_pos = draggable_boundary.height()
	    random_obj = [];
	    d = [{x: 10,y: 380}]
		if (!isObjAlready) {
	    
	    draggable_objects[0].innerHTML = "";
    for (i=1; i<=obj_no; i++){
	random_obj[i-1] = $("<div id=\"random_obj" + i + "\" class=\"random_obj\" ></div>");
	draggable_objects.append(random_obj[i-1]);
	var c,h;
		t = random_obj[i-1];
		    x2 = t.width();
		    y2 = t.height();
		    var tries = 0;
		    do{
			x1 = Math.random()*(1024-130);
			y1 = Math.random()*(600-130);
			tries++;
		    }
		    while (isOverlaped(x1, y1, x2, y2) && tries<=3000)
		    ObjPos.push({x:x1, y:y1, w:x2, h:y2});
		    t.css({'position':'absolute','cursor':'pointer','top':''+y1+'px','left':''+x1+'px'})	    
		
		}
		
		var numArr=[1,2,3,4,5,6,7,8,9];
		
		var total = numArr[Math.floor(Math.random()*numArr.length)];
		
		
		console.log(total)
		for (var i=1;i<=total;i++) {
			$('#random_obj'+i).css({'background-image':'url(./images/'+randomObj1+'.png)','background-size': 'contain','background-repeat': 'no-repeat'});
		}	
		
		for (var i=(total+1);i<=10;i++) {
			$('#random_obj'+i).css({'background-image':'url(./images/'+randomObj2+'.png)','background-size': 'contain','background-repeat': 'no-repeat'});
		}
	}
	    
		var nn = draggable_objects.children().length;
		//console.log(nn,"ask")
		for (i=0; i<nn; i++){
		    random_obj[i] = draggable_objects.children().eq(i);
		    random_obj[i].OrgX = random_obj[i].position().left;
		    random_obj[i].OrgY = random_obj[i].position().top;
		    random_obj[i].draggable({
			containment:'#draggable_boundary',
			stack:$(".random_obj"),
			start:function(event,ui) {
				draggingEnabled = true;
				$('.icoReset').css({'opacity':'1', 'cursor':'pointer'});
				$('.icoReset').bind('click', resetfn);
			}		
			});
		}
	}
    }
    function isOverlaped(x1, y1, x2, y2){
    var isOverlaped = false;
    for(var item=0;item<ObjPos.length;item++){
     if((x1 > ObjPos[item].x && x1 < (ObjPos[item].x+ObjPos[item].w)) && (y1 < ObjPos[item].y && (y1+y2) > (ObjPos[item].y))){
	isOverlaped = true;
	}
	if((x1 < ObjPos[item].x && (x1+x2) > ObjPos[item].x) && (y1 < ObjPos[item].y && (y1+y2) > (ObjPos[item].y))){
	isOverlaped = true;
	}
	if((x1 < ObjPos[item].x && (x1+x2) > ObjPos[item].x) && (y1 > ObjPos[item].y && (y1+y2) > (ObjPos[item].y+ObjPos[item].h))){
	isOverlaped = true;
	}
	if((x1 > ObjPos[item].x && x1 < ObjPos[item].x+ObjPos[item].w) && (y1 > ObjPos[item].y && y1 < (ObjPos[item].y+ObjPos[item].h))){
	isOverlaped = true;
	}
	if((x1 > ObjPos[item].x && x1+x2 < ObjPos[item].x+ObjPos[item].w) && (y1 < ObjPos[item].y && (y1+y2) > ObjPos[item].y)){
	isOverlaped = true;
	}
	if((x1 < ObjPos[item].x && x1+x2 > ObjPos[item].x) && (y1 > ObjPos[item].y && (y1+y2) < ObjPos[item].y+ObjPos[item].h)){
	isOverlaped = true;
	}
	if((x1 > ObjPos[item].x && x1+x2 < ObjPos[item].x+ObjPos[item].w) && (y1 < ObjPos[item].y+ObjPos[item].h && (y1+y2) > ObjPos[item].y+ObjPos[item].h)){
	isOverlaped = true;
	}
	if((x1 > ObjPos[item].x && x1 < ObjPos[item].x+ObjPos[item].w) && (y1 > ObjPos[item].y && y1 < ObjPos[item].y+ObjPos[item].h)){
	isOverlaped = true;
	}
	if((x1 > ObjPos[item].x && x1 < ObjPos[item].x+ObjPos[item].w && x1+x2>ObjPos[item].x && x1+x2<ObjPos[item].x+ObjPos[item].w) && (y1 > ObjPos[item].y && y1 < ObjPos[item].y+ObjPos[item].h && y1+y2>ObjPos[item].y && y1+y2<ObjPos[item].y+ObjPos[item].h)){
	isOverlaped = true;
	}
   }
	return isOverlaped;
}
    function shuffle(o)
	{ 
	    for(var j, x, i = o.length; i; j = Math.floor(Math.random() * i), x = o[--i], o[i] = o[j], o[j] = x);
	    return o;
	}
