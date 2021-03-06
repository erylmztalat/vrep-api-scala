# v-rep scala
Simple Scala binding for
[Coppelia Robotics V-REP simulator](http://www.coppeliarobotics.com/) ([remote API](http://www.coppeliarobotics.com/helpFiles/en/remoteApiOverview.htm))

## Getting started
1. Add the library in your sbt project by simply adding the following dependency to your build file:
```scala
libraryDependencies += "com.github.troxid" %% "vrepapiscala" % "0.2.1"
```
2. Copy platform-specific native library from
    `V-REP/programming/remoteApiBindings/java/lib/`
    into the root folder of your project (or configure `java.library.path`)
3. Find the socket port number in `V-REP/remoteApiConnections.txt` and use
    it to connect to the server simulator

## Currently implemented things
In the current version is not implemented features such as remote management GUI,
additional configuration properties of objects and shapes, etc.
Basically implemented those components that are required to control the robot:
* Joint
* Proximity sensor
* Vision sensor
* Force sensor
* Position sensor (used for that dummy or shape object)
* Remote function calls

## Example
```scala
import vrepapiscala._

object PioneerRemoteControlExample extends App {
  val api = VRepAPI.connect("127.0.0.1", 19997).get
  val robot = new PioneerP3dx(api)

  api.simulation.start()

  for(_ <- 0 to 500){
    val resLS = robot.leftSensor.read
    val resRS = robot.rightSensor.read
    if(resLS.detectionState && resLS.detectedPoint.length < 0.5){
      robot.rotateRight()
    }else if(resRS.detectionState && resRS.detectedPoint.length < 0.5){
      robot.rotateLeft()
    } else {
      robot.moveForward()
    }
    Thread.sleep(10)
  }

  api.simulation.stop()
}

class PioneerP3dx(api: VRepAPI) {
  private val speed = 2f
  private val leftMotor = api.joint.withVelocityControl("Pioneer_p3dx_leftMotor").get
  private val rightMotor = api.joint.withVelocityControl("Pioneer_p3dx_rightMotor").get
  private val frontSensors =
    for(i <- 1 to 8)
      yield api.sensor.proximity("Pioneer_p3dx_ultrasonicSensor" + i).get

  def moveForward(): Unit = {
    leftMotor.setTargetVelocity(speed)
    rightMotor.setTargetVelocity(speed)
  }

  def moveBackward(): Unit = {
    leftMotor.setTargetVelocity(-speed)
    rightMotor.setTargetVelocity(-speed)
  }

  def rotateLeft(): Unit = {
    leftMotor.setTargetVelocity(-speed)
    rightMotor.setTargetVelocity(speed)
  }

  def rotateRight(): Unit = {
    leftMotor.setTargetVelocity(speed)
    rightMotor.setTargetVelocity(-speed)
  }

  def stop(): Unit = {
    leftMotor.setTargetVelocity(0)
    rightMotor.setTargetVelocity(0)
  }

  def leftSensor = frontSensors(1)

  def rightSensor = frontSensors(6)
}
```


