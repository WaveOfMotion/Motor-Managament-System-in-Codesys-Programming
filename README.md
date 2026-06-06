This PLC-based logic of multiple motor start/stop is developped on function block FB_Motor defined variables :
(.. Since Methods are like Functions of function block, I used them more to not store everything inside programs or function blocks ..)

1) Counts how many times motor has been started :

  TrigCount(CLK := MotorRunning);

  IF TrigCount.Q THEN
    iStartCount := iStartCount + 1;
  END_IF

2) Total working time :

  Timer(In := MotorRunning, PT := T#24H);

  IF MotorRunning THEN
	  RunningTime := RunningTime + (Timer.ET - LastET); 
  END_IF

  LastET := Timer.ET;

  IF NOT MotorRunning THEN
	  LastET := T#0S; (* Reset *)
  END_IF

Then I introduce manual start and 4 methods:
  
1) Start motors starting with the motor with least starts - Start_By_Min_iCount,
2) Stop motors starting with the motor with the most starts - Stop_By_Max_iCount,
3) Start motors starting withh the motor with least workinng time - Start_By_Min_Tsum,
4) Stop motors starting with the motor with most workingtime - Start_By_Max_Tsum.

!!! Also an important thing to notice: Since I have for instance only one 24V DI (digital input) for starting the motor, it should be controlled so that if manual start
is HIGH, motor starts, if LOW motor is stops. But I have also 4 buttons as mentioned before. If the "Start/Stop selection methods" ae used, they have to override
manual start, and the otherway around. So I introduce in my MotorLogic one cooll thing - PRIORITY:

FOR n := 1 TO 10 DO

  IF Priority = 1 THEN
    MotorTree.MotorStart[n] := ManualStart[n];
    ModeStart[n] := ManualStart[n];
  END_IF

  IF Priority = 2 THEN
    MotorTree.MotorStart[n] := ModeStart[n];
    ManualStart[n] := ModeStart[n];


In conclusion I can say,  after implementing this, I can manually start - SWITCH PRIORITY, and motors HOLDS their states. 
Then define how much motors to stop, and stop by 4 selections, respective to motor given statistics.

Important note here: If you want to extend the control system, you can replace Motor Counts with Motor Rotation or Motor Temperature.
The logic itself just proved that it can sort array in ascending/descending motor start/stop order respective to the data provided, for this project - iCounts, Tsum. 



