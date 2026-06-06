# Motor START/STOP Management System with selection

In short-introduction, this PLC-based logic of multiple motor start/stop is developped on function block FB_Motor defined variables :

1) Counts how many times motor has been started :
```iecst
TrigCount(CLK := MotorRunning);

IF TrigCount.Q THEN
    iStartCount := iStartCount + 1;
END_IF
```
2) Total working time :
```iecst
Timer(In := MotorRunning, PT := T#24H);

IF MotorRunning THEN
	RunningTime := RunningTime + (Timer.ET - LastET); 
END_IF

LastET := Timer.ET;

IF NOT MotorRunning THEN
	LastET := T#0S; (* Reset *)
END_IF
```
Then I introduce manual start and 4 methods:
  
1) Start motors starting with the motor with least starts - Start_By_Min_iCount :
```iecst
FOR j := 1 TO 10 DO
			
		iMin_C := 16#7FFFFFFF; (* Max value of DINT *)
		iMinIndex_C := 0;
		
		(* Build MotorOrder storing first element with LEAST COUNTS *)
		FOR i := 1 TO 10 DO

			IF iCount[i] < iMin_C THEN
				iMin_C := iCount[i];
				iMinIndex_C := i;
			END_IF	
		END_FOR
		
		MotorOrder[j] := iMinIndex_C;
		iCount[iMinIndex_C] := 16#7FFFFFFF; (* Max value of DINT *)
END_FOR
```
2) Stop motors starting with the motor with the most starts - Stop_By_Max_iCount :
```iecst
FOR j := 1 TO 10 DO
			
		iMax_C := 0; (* Min value of DINT *)
		iMaxIndex_C := 0;
		
		(* Build MotorOrder storing first element with LEAST COUNTS *)
		FOR i := 1 TO 10 DO

			IF iCount[i] > iMax_C THEN
				iMax_C := iCount[i];
				iMaxIndex_C := i;
			END_IF	
		END_FOR
		
		MotorOrder[j] := iMaxIndex_C;
		iCount[iMaxIndex_C] := 0; (* Min value of DINT *)
END_FOR
```
3) Start motors starting withh the motor with least workinng time - Start_By_Min_Tsum :
```iecst
FOR j := 1 TO 10 DO
		
		iMin_T := 3.402823E+38; (* Max value of REAL *)
		iMinTime_T := 0;
		
		(* Build MotorOrder storing first element with shortest working time Tsum *)
		FOR i := 1 TO 10 DO
			
			IF iTsum[i] < iMin_T THEN
				iMin_T := iTsum[i];
				iMinTime_T := i;
			END_IF
		END_FOR
		
		MotorOrder[j] := iMinTime_T;
		iTsum[iMinTime_T] := 3.402823E+38; (* Max value of TIME *)
END_FOR
```
4) Stop motors starting with the motor with most workingtime - Start_By_Max_Tsum :
```iecst
FOR j := 1 TO 10 DO
		
		iMax_T := 0; (* Min value of TIME *)
		iMaxTime_T := 0;
		
		(* Build MotorOrder storing first element with shortest working time Tsum *)
		FOR i := 1 TO 10 DO
			
			IF iTsum[i] > iMax_T THEN
				iMax_T := iTsum[i];
				iMaxTime_T := i;
			END_IF
		END_FOR
		
		MotorOrder[j] := iMaxTime_T;
		iTsum[iMaxTime_T] := 0; (* Min value of TIME *)
END_FOR
```

### Also an important thing to notice: 
Since I have 1 x 24V DI (digital input) for starting the motor, 
it should be controlled so that if manual start is HIGH, motor starts, if LOW motor stops. Everything simple.
But I have also 4 buttons as mentioned before. If the "Start/Stop selection methods" are used, 
they have to override manual start, and the otherway around.
So I implement priority switching. And provide that priority switching doesn't affect motor state:
```iecst
FOR n := 1 TO 10 DO
	
	(* IF Priority := 1 >>>>>> Manual Mode *)
	IF Priority = 1 THEN
		MotorTree.MotorStart[n] := ManualStart[n];
		ModeStart[n] := ManualStart[n];
	END_IF
	
	(* IF Priority := 2 >>>>>> Mode Selection *)
	IF Priority = 2 THEN
		MotorTree.MotorStart[n] := ModeStart[n];
		ManualStart[n] := ModeStart[n];
	END_IF
	
END_FOR
```
## Conclusion
if you want to extend the control system, 
you can replace Motor Counts with Motor Rotation or Motor Temperature.
The logic itself just proved that it can sort array in ascending/descending motor start/stop order, 
respective to the data provided, and for this project - iCounts, Tsum. 
