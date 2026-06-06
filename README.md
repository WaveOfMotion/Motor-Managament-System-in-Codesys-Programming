This PLC-based logic of multiple motor start/stop is developped on function block FB_Motor defined variables :
(.. Since Methods are like Functions of function block, I used them more to not store everything inside programs or function blocks ..)

1) Counts how many times motor has been started,
2) Total working time.

Then I introduce manual start and 4 methods:
  
1) Start motors starting with the motor with least starts - Start_By_Min_iCount,
2) Stop motors starting with the motor with the most starts - Stop_By_Max_iCount,
3) Start motors starting withh the motor with least workinng time - Start_By_Min_Tsum,
4) Stop motors starting with the motor with most workingtime - Start_By_Max_Tsum.

!!! Also an important thing to notice: Since I have for instance only one 24V DI (digital input) for starting the motor, it should be controlled so that if manual start
is HIGH, motor starts, if LOW motor is stops. But I have also 4 buttons as mentioned before. If the "Start/Stop selection methods" ae used, they have to override
manual start, and the otherway around. So I implement priority switching. And if priority is changed, motor holds the last value.


Last words being said, if you want to extend the control system, you can replace Motor Counts with Motor Rotation or Motor Temperature.
The logic itself just proved that it can sort array in ascending/descending motor start/stop order respective to the data provided, for this project - iCounts, Tsum. 
