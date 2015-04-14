//Pool by John Bezanis for swfspot.com


inputFriction.text = "0.015"
inputMass.text = "170"
//Conversion for degrees to radians, calculated once for increased performance
degreestoradians = Math.PI/180;
//total balls sunk
ballssunk = 0;
//Amount of space in between balls when racked and in the tray
spacer = .2;

//Determine the interval of time between checking the ball movement/collisions in milliseconds
//a lower number gives better accuracy but is more cpu intense
//if this number is adjusted, friction and powermultiplier should also be adjusted.
movetime=10;
//Define a function to shuffle an array
Array.prototype.shuffle = function() {
	for (i=0; i<this.length; i++) {
		this.push(this.splice(Math.floor(Math.random()*this.length), 1));
	}
};

//Main function that adds the balls to the table
function newGame() {
	//Set the position of the rack
	rackx = tablearea._x+tablearea._width*.75;
	racky = tablearea._y+tablearea._height/2;
	//attach the 8 ball at the center of the rack
	this.attachMovie('balls', 'ball8', this.getNextHighestDepth(), {_x:rackx, _y:racky});
	//go to the frame with the 8 ball graphic
	ball8.gotoAndStop(8);
	//initialize the x and y veocity to 0
	ball8.xvel = 0;
	ball8.yvel = 0;
	//set the ball to not sunk
	ball8.sunk = 0;
	//create an array with all of the other balls
	ballArray = [1, 2, 3, 4, 5, 6, 7, 9, 10, 11, 12, 13, 14, 15];
	//shuffle the array, shuffling the position of the balls
	ballArray.shuffle();
	//calculate the horizontal difference between two rows of balls
	horizdiff = Math.sqrt(Math.pow(ball8._width+spacer, 2)-Math.pow((ball8._height+spacer)/2, 2));
	//this array contains the coordinates of the balls within the rack
	ballcoords = [[rackx-horizdiff*2, racky], [rackx-horizdiff, racky-(ball8._height+spacer)/2], [rackx-horizdiff, racky+(ball8._height+spacer)/2], [rackx, racky-(ball8._height+spacer)], [rackx, racky+(ball8._height+spacer)], [rackx+horizdiff, racky-(ball8._height+spacer)*1.5], [rackx+horizdiff, racky-(ball8._height+spacer)*.5], [rackx+horizdiff, racky+(ball8._height+spacer)*.5], [rackx+horizdiff, racky+(ball8._height+spacer)*1.5], [rackx+horizdiff*2, racky-(ball8._height+spacer)*2], [rackx+horizdiff*2, racky-(ball8._height+spacer)], [rackx+horizdiff*2, racky], [rackx+horizdiff*2, racky+(ball8._height+spacer)], [rackx+horizdiff*2, racky+(ball8._height+spacer)*2]];
	//loop through all of the balls, creating them and positioning them in the rack
	for (ballpos=0; ballpos<14; ballpos++) {
		this.attachMovie('balls', 'ball'+ballArray[ballpos], this.getNextHighestDepth(), {_x:ballcoords[ballpos][0], _y:ballcoords[ballpos][1]});
		//initialize the x and y velocity to 0
		eval('ball'+ballArray[ballpos]).xvel = 0;
		eval('ball'+ballArray[ballpos]).yvel = 0;
		//set the ball to not sunk
		eval('ball'+ballArray[ballpos]).sunk = 0;
		//change the graphic to represent the ball
		eval('ball'+ballArray[ballpos]).gotoAndStop(ballArray[ballpos]);
	}
	//attach the cue ball
	this.attachMovie('balls', 'ball0', this.getNextHighestDepth(), {_x:tablearea._x+tablearea._width/4, _y:tablearea._y+tablearea._height/2});
	//frame 16 is thge cue ball frame
	ball0.gotoAndStop(16);
	//allow the user to move the cue ball around the left quarter of the table
	scratch();
	//process the balls on the table on each interval, which its length is determined by 'movetime'
	var ballinterval:Number = setInterval(this, "moveBalls", movetime);
}

function moveBalls(){
		//loop through each ball, moving it according to its velocity
		for (curball=0; curball<=15; curball++) {
			//if the ball is sunk, move it along the bottom tray
			if (eval('ball'+curball).sunk) {
				//move the ball towards the right of the tray until it hits the end or another ball
				eval('ball'+curball)._x = Math.min(trayarea._x+trayarea._width-eval('ball'+curball)._width/2-eval('ball'+curball)._width*(eval('ball'+curball).sunk-1)-spacer*(eval('ball'+curball).sunk-1), eval('ball'+curball)._x+5);
			} else {
				//move the ball according to its velocity
				eval('ball'+curball)._x += eval('ball'+curball).xvel;
				eval('ball'+curball)._y += eval('ball'+curball).yvel;
				//decrease the velocity to account for 
				eval('ball'+curball).xvel -= eval('ball'+curball).xvel*friction;
				eval('ball'+curball).yvel -= eval('ball'+curball).yvel*friction;
				//if the ball is moving very slowly, stop its movement completely
				if (Math.abs(eval('ball'+curball).xvel)<.02&&Math.abs(eval('ball'+curball).yvel)<.02) {
					eval('ball'+curball).xvel = 0;
					eval('ball'+curball).yvel = 0;
				}
				//Ball bounces off of the right bumper
				if (eval('ball'+curball)._x+eval('ball'+curball)._width/2>tablearea._x+tablearea._width) {
					//if the ball is high enough or low enough to fall into a pocket, sink it
					if (eval('ball'+curball)._y-eval('ball'+curball)._height*.75<tablearea._y || eval('ball'+curball)._y+eval('ball'+curball)._height*.75>tablearea._y+tablearea._height) {
						sinkBall(curball);
					} else {
						//reverse the velocity and adjust for the distance past the bumper
						eval('ball'+curball).xvel *= -1;
						eval('ball'+curball)._x -= (eval('ball'+curball)._x+eval('ball'+curball)._width/2-(tablearea._x+tablearea._width));
					}
				//ball bounces off of the left bumper
				} else if (eval('ball'+curball)._x-eval('ball'+curball)._width/2<tablearea._x) {
					//if the ball is high enough or low enough to fall into a pocket, sink it
					if (eval('ball'+curball)._y-eval('ball'+curball)._height*.75<tablearea._y || eval('ball'+curball)._y+eval('ball'+curball)._height*.75>tablearea._y+tablearea._height) {
						sinkBall(curball);
					} else {
						//reverse the velocity and adjust for the distance past the bumper
						eval('ball'+curball).xvel *= -1;
						eval('ball'+curball)._x -= (eval('ball'+curball)._x-eval('ball'+curball)._width/2)-tablearea._x;
					}
				}
				//ball bounces off of the bottom bumper
				if (eval('ball'+curball)._y+eval('ball'+curball)._height/2>tablearea._y+tablearea._height) {
					//if the ball is left or right enough to fall into a pocket, or in the center pocket area, sink it
					if (eval('ball'+curball)._x-eval('ball'+curball)._width*.75<tablearea._x || eval('ball'+curball)._x+eval('ball'+curball)._width*.75>tablearea._x+tablearea._width || (eval('ball'+curball)._x-eval('ball'+curball)._width*.75<tablearea._x+tablearea._width/2 && eval('ball'+curball)._x+eval('ball'+curball)._width*.75>tablearea._x+tablearea._width/2)) {
						sinkBall(curball);
					} else {
						//reverse the velocity and adjust for the distance past the bumper
						eval('ball'+curball).yvel *= -1;
						eval('ball'+curball)._y -= (eval('ball'+curball)._y+eval('ball'+curball)._height/2-(tablearea._y+tablearea._height));
					}
				//ball bounces off of the top bumper	
				} else if (eval('ball'+curball)._y-eval('ball'+curball)._height/2<tablearea._y) {
					//if the ball is left or right enough to fall into a pocket, or in the center pocket area, sink it
					if (eval('ball'+curball)._x-eval('ball'+curball)._width*.75<tablearea._x || eval('ball'+curball)._x+eval('ball'+curball)._width*.75>tablearea._x+tablearea._width || (eval('ball'+curball)._x-eval('ball'+curball)._width*.75<tablearea._x+tablearea._width/2 && eval('ball'+curball)._x+eval('ball'+curball)._width*.75>tablearea._x+tablearea._width/2)) {
						sinkBall(curball);
					} else {
						//reverse the velocity and adjust for the distance past the bumper
						eval('ball'+curball).yvel *= -1;
						eval('ball'+curball)._y -= (eval('ball'+curball)._y-eval('ball'+curball)._height/2)-tablearea._y;
					}
				}
			}
		}
		//if the cue ball is on the table, check for ball collisions
		if (!ball0.onPress) {
			//Check the distances between balls
			for (i=0; i<=15; i++) {
				a = this["ball"+i];
				for (j=i+1; j<=15; j++) {
					b = this["ball"+j];
					//determine the distance on the x and y planes
					var dx = b._x-a._x;
					var dy = b._y-a._y;
					//use the pythagorean theorem to determine the distance between two balls
					var dist = Math.sqrt(dx*dx+dy*dy);
					//if there is a collision between two balls, process it.
					if (dist<a._width/2+b._width/2) {
						_root.solveBalls(a, b);
					}
				}
			}
		}
};

function solveBalls(ballA, ballB)
{

    //find the distance between the two balls in their x and y components
    var dx = ballB._x - ballA._x;
    var dy = ballB._y - ballA._y;

    //find angle of collision in radians
    var collisionAngle = Math.atan2(dy, dx);

    //find magnitude of velocity vector for each ball
    var speedA = Math.sqrt(ballA.xvel * ballA.xvel + ballA.yvel * ballA.yvel);
    var speedB = Math.sqrt(ballB.xvel * ballB.xvel + ballB.yvel * ballB.yvel);

    //find direction of travel for each ball in radians
    var directionA = Math.atan2(ballA.yvel, ballA.xvel);
    var directionB = Math.atan2(ballB.yvel, ballB.xvel);

    //redefine the velocity vectors in a new coordinate system aligned with the collision angle
    var xcollisionvelocityA = speedA * Math.cos(directionA - collisionAngle);
    var ycollisionvelocityA = speedA * Math.sin(directionA - collisionAngle);
    var xcollisionvelocityB = speedB * Math.cos(directionB - collisionAngle);
    var ycollisionvelocityB = speedB * Math.sin(directionB - collisionAngle);

    //solve for new velocities following the collision using conservation of momentum laws
    //yvelocities stay the same
    var NewxcollisionvelocityA = xcollisionvelocityB;
    var NewxcollisionvelocityB = xcollisionvelocityA;

    //redefine after-collision velocities to original coordinate system
    ballA.xvel = Math.cos(collisionAngle) * NewxcollisionvelocityA + Math.cos(collisionAngle + Math.PI / 2) * ycollisionvelocityA;
    ballA.yvel = Math.sin(collisionAngle) * NewxcollisionvelocityA + Math.sin(collisionAngle + Math.PI / 2) * ycollisionvelocityA;
    ballB.xvel = Math.cos(collisionAngle) * NewxcollisionvelocityB + Math.cos(collisionAngle + Math.PI / 2) * ycollisionvelocityB;
    ballB.yvel = Math.sin(collisionAngle) * NewxcollisionvelocityB + Math.sin(collisionAngle + Math.PI / 2) * ycollisionvelocityB;

    //update the positions of the balls to make sure they don't overlap
    var dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < ballA._width / 2 + ballB._width / 2)
    {
        var DesiredDist = ballA._width / 2 + ballB._width / 2;
        var OverlapFactor = (DesiredDist / dist);
        ballA._x -= ((dx * OverlapFactor)-dx)/2;
        ballA._y -= ((dy * OverlapFactor)-dy)/2;
        ballB._x += ((dx * OverlapFactor)-dx)/2;
        ballB._y += ((dy * OverlapFactor)-dy)/2;
    }

}

//sink a ball when it hits a pocket
function sinkBall(curball) {
	//if the sunk ball is the cue ball, it causes a scratch
	if (curball == 0) {
		//bring the cue ball back up and let the player move it around
		scratch();
	//else it isn't the cue ball, so move the ball down to the bottom tray
	} else {
		//make sure the ball isn't already sunk
		if (eval('ball'+curball).sunk == 0) {
			//enumerate the number of balls sunk and give this ball that value
			//the value is used to determine how far to the right the ball rolls
			eval('ball'+curball).sunk = (++ballssunk);
			//stop the ball's movement. We will manually move it to the right
			eval('ball'+curball).xvel = 0;
			eval('ball'+curball).yvel = 0;
			//move the ball down into the tray
			eval('ball'+curball)._y = trayarea._y+trayarea._height/2;
			//move the ball to the left side of the tray
			eval('ball'+curball)._x = trayarea._x+eval('ball'+curball)._width/2;
		}
	}
}
//
function scratch() {
	//allow the player to drag the cue ball around the left quarter of the table
	//spacer gives a buffer so the ball will not go in a hole when it is being drug around
	ball0.startDrag(true, tablearea._x+ball0._width/2+spacer, tablearea._y+ball0._height/2+spacer, tablearea._x+tablearea._width/4, tablearea._y+tablearea._height-ball0._height/2-spacer);
	//reset the speed of the cue ball to 0
	ball0.xvel = 0;
	ball0.yvel = 0;
	//the ball has been pressed, place it on the table and show the pool stick
	ball0.onPress = function() {
		//Check the distances between balls
		a = this;
		//don't drop the ball if it is touching another ball
		for (j=1; j<=15; j++) {
			b = this._parent["ball"+j];
			var dx = b._x-a._x;
			var dy = b._y-a._y;
			//determine the distance between 2 balls
			var dist = Math.sqrt(dx*dx+dy*dy);
			if (dist<a._width/2+b._width/2) {
				return;
			}
		}
		//The ball is not touching any othe balls, so drop it on the table
		ball0.stopDrag();
		//show the pool stick
		showStick();
		//delete this function so the player cannot click the ball anymore
		delete ball0.onPress;
	};
}
//Show the pool stick
function showStick() {
	//if a pool stick already exists, delete it
	if (poolstick) {
		delete poolstick;
	}
	//create a new poolstick
	this.attachMovie('poolstick', 'poolstick', this.getNextHighestDepth(), {_x:ball0._x, _y:ball0._y});
	//initialize the power to 0
	poolstick.power = 0;
	//initialize the distance to pull back the stick to 0
	poolstick.curdist = 0;
	//on each frame move the stick according to the mouse and ball position 
	poolstick.onEnterFrame = function() {
		//if the pool stick isn't fading out (it hasn't been clicked and released) move it according to the ball's position 
		if (poolstick._alpha == 100) {
			poolstick._x = ball0._x;
			poolstick._y = ball0._y;
		}
		//If the poolstick hasn't been clicked yet, adjust its angle based on the mouse's angle relative to the cue ball
		if (poolstick.onPress) {
			//set the angle
			stickangle = Math.atan(-(_ymouse-ball0._y)/(_xmouse-ball0._x))/(Math.PI/180);
			if ((_xmouse-ball0._x)<0) {
				stickangle += 180;
			}
			if ((_xmouse-ball0._x)>=0 && (-(_ymouse-ball0._y))<0) {
				stickangle += 360;
			}
			poolstick._rotation = -stickangle-90;
		//else the pool stick has been clicked, so keep its angle locked from when it was clicked	
		} else {
			//when the pool stick is clicked, allow the player to bring it back, giving it power
			if (poolstick.onRelease) {
				//the power is based on the distance the mouse is from the ball relative to when it was first clicked
				//keep the power stored for when the stick is released
				//set a cap to how far away from the ball the stick can go
				poolstick.power = Math.min(100, Math.max(0, (Math.sqrt(Math.pow(_xmouse-ball0._x, 2)+Math.pow(_ymouse-ball0._y, 2))-poolstick.clickdistance)));
				//set the distance equal to the amout of power applied
				poolstick.curdist = poolstick.power;
			//else the pool stick has been released. Start bringing the stick towards the ball
			} else {
				//move the stick towards the ball
				poolstick.curdist = Math.max(0, poolstick.curdist-20);
				//if there is no more distance to get to the ball, hit it with the stick
				if (poolstick.curdist == 0) {
					//if this is the first frame the poolstick  has hit the ball, change the ball's velocity
					if (poolstick._alpha == 100) {
						//hit the ball in the direction the pool stick is pointed
						ball0.xvel = Math.sin(poolstick._rotation*degreestoradians)*(poolstick.power*powermultiplier);
						ball0.yvel = -Math.cos(poolstick._rotation*degreestoradians)*(poolstick.power*powermultiplier);
					}
					//start fading out the pool stick
					poolstick._alpha -= 10;
					//if the pool stick is fully invisible, delete it and set an interval that waits until all balls have stopped moving 
					if (poolstick._alpha<=0) {
						//called every 30 milliseconds
						setMovementInterval();
						removeMovieClip(poolstick);
					}
				}
			}
		}
		//if the poolstick hasn't been released or clicked yet, move it to the cue ball's edge
		if (poolstick._alpha == 100) {
			//We figure out the direction the poolstick is pointing, and then move it away from the ball according to its angle
			poolstick._x += -Math.sin(degreestoradians*poolstick._rotation)*(ball0._width/2+poolstick.curdist);
			poolstick._y += Math.cos(degreestoradians*poolstick._rotation)*(ball0._width/2+poolstick.curdist);
		}
	};
	//adds the function to drag the poolstick when it is clicked
	poolstickAction();
}
//add a listener to the poolstick to start dragging when it is clicked
function poolstickAction() {
	//the listener that waits for the poolstick to be clicked
	poolstick.onPress = function() {
		//determine the distance of the mouse from the ball, and store it
		poolstick.clickdistance = Math.sqrt(Math.pow(_xmouse-ball0._x, 2)+Math.pow(_ymouse-ball0._y, 2));
		//add a listener that waits for the user to release the mouse
		poolstick.onRelease = poolstick.onReleaseOutside=function () {
			//get rid of the current poolstick listeners
			delete poolstick.onRelease;
			delete poolstick.onReleaseOutside;
			//if the poolstick is not pulled back, recreate the 
			if (poolstick.power<=0) {
				//else the poolstick is not pulled back, so recreate the poolstick action
				poolstickAction();
			}
		};
		//delete the poolstick press listener
		delete poolstick.onPress;
	};
}
//create an interval to check if any balls have moved since the last interval
function setMovementInterval() {
	//run the interval every 'movetime' milliseconds
	var movementinterval:Number = setInterval(this, "checkMovement", movetime);
}
//function that is called by the interval
function checkMovement():Void {
	//if the poolstick is still on the table, do not check if the balls have stopped moving
	if (poolstick) {
		return;
	}
	//loop through all of the balls to check for movement
	for (i=0; i<=15; i++) {
		if (eval('ball'+i).xvel != 0 || eval('ball'+i).yvel != 0) {
			return;
		}
	}
	//if the user hasn't scratched, show the poolstick
	if (!ball0.onPress) {
		showStick();
	}
}
//function called when the newgame button is pressed, aka rack 'em
newgame.onPress = function() {
	//Rolling friction: pool ball on felt has a minimum coefficient of 0.015
	friction = parseFloat(inputFriction.text);
	mass = parseFloat(inputMass.text) //170g is regulation ball weight
	//The effect of the stick is inversely proportional to the ball's mass
	powermultiplier = 40/mass;
	//if ball0 exists, all of the balls exist, so we first delete all of the balls
	if (ball0) {
		for (i=0; i<=15; i++) {
			//delete all of the balls
			removeMovieClip('ball'+i);
		}
		//delete the main function that runs at the start of each frame
		delete onEnterFrame;
	}
	//if the poolstick exists, delete it
	if (poolstick) {
		removeMovieClip('poolstick');
	}
	//start a new game
	newGame();
};
