# arduinoSimGpsTracker
Arduino GPS Tracker using module sim808 evb v3


This is a project that can be useful to keep location of specific object on command, it can easily be adapted to different purposes like rando/bike tracker in the aim to see the itinary that has been taken by the tracker carrier.

The use of the sim 808 give to this project a lot of possibility  for evolving as it allows to interract with a sim card to send/receive calls and sms, and it allows to interract with GPS at the same time which can be very usefull with a lot of various uses.

It can be use as follow : you send a specific sms to the module and it sends you back a sms with its current location with 2 links to  websites that show the location on a map.

At this moment this project is static, but it can be linked to a vehicule battery with a little adaptation in the aim to be power supplied. It could also carry portative batteries or a combination of both: the vehicule load the embedded batteries and if the vehicule battery is removed, then the internal batteries are able to power supply the module that is then still able to send its location... many scenarios are able to be thought.


It uses:
hardware:
- A chinese arduino uno replica called "geekreit" that can easily be found at a very abordable price (4$ on website like bangood)
- A sim 808 module  available for a few more boxes on ebay: https://www.ebay.fr/itm/193455526273 (SIM808 Module GSM GPRS GPS Development Board SMA avec antenne GPS pour Arduino)

software:
the DFRobot_sim808 library (https://github.com/DFRobot/DFRobot_SIM808)









=====================================================================================
/!\Carefull: the library default code is inaccurate with the gps location so you can replace the function getGPS() in DFRobot_sim808.cpp
by the following one:

 bool DFRobot_SIM808::getGPS()
{
	if(!getGPRMC()) //没有得到$GPRMC字符串开头的GPS信息
	return false;
	// Serial.println(receivedStack);
	if(!parseGPRMC(receivedStack)) //不是$GPRMC字符串开头的GPS信息
	return false;

	// skip mode
	char *tok = strtok(receivedStack, ","); //起始引导符
	if (! tok) return false;

	// grab time //<1> UTC时间，格式为hhmmss.sss；
	// tok = strtok(NULL, ",");
	char *time = strtok(NULL, ",");
	if (! time) return false;
	uint32_t newTime = (uint32_t)parseDecimal(time);
	getTime(newTime);

	// skip fix
	tok = strtok(NULL, ","); //<2> 定位状态，A=有效定位，V=无效定位
	if (! tok) return false;

	// grab the latitude
	char *latp = strtok(NULL, ","); //<3> 纬度ddmm.mmmm(度分)格式(前面的0也将被传输)
	if (! latp) return false;

	// grab latitude direction // <4> 纬度半球N(北半球)或S(南半球)
	char *latdir = strtok(NULL, ",");
	if (! latdir) return false;

	// grab longitude //<5> 经度dddmm.mmmm(度分)格式(前面的0也将被传输)
	char *longp = strtok(NULL, ",");
	if (! longp) return false;

	// grab longitude direction //<6> 经度半球E(东经)或W(西经)
	char *longdir = strtok(NULL, ",");
	if (! longdir) return false;

	double latitude = atof(latp);
	double longitude = atof(longp);

	// convert latitude from minutes to decimal
	double degrees = floor(latitude / 100);
	double minutes = latitude - (100 * degrees);
	minutes /= 60;
	degrees += minutes;

	// turn direction into + or -
	if (latdir[0] == 'S') degrees *= -1;

	//*lat = degrees;
	GPSdata.lat = degrees;

	// convert longitude from minutes to decimal
	degrees = floor(longitude / 100);
	minutes = longitude - (100 * degrees);
	minutes /= 60;
	degrees += minutes;


	// turn direction into + or -
	if (longdir[0] == 'W') degrees *= -1;

	//*lon = degrees;
	GPSdata.lon= degrees;

	// only grab speed if needed //<7> 地面速率(000.0~999.9节，前面的0也将被传输)
	// if (speed_kph != NULL) {

	// grab the speed in knots
	char *speedp = strtok(NULL, ",");
	if (! speedp) return false;

	// convert to kph
	//*speed_kph = atof(speedp) * 1.852;
	GPSdata.speed_kph= atof(speedp) * 1.852;

	// }

	// only grab heading if needed //地面航向(000.0~359.9度，以真北为参考基准，前面的0也将被传输)
	// if (heading != NULL) {

	// grab the speed in knots
	char *coursep = strtok(NULL, ",");
	if (! coursep) return false;

	//*heading = atof(coursep);
	GPSdata.heading = atof(coursep);
	// }

	// grab date
	char *date = strtok(NULL, ","); //<3> 纬度ddmm.mmmm(度分)格式(前面的0也将被传输)
	if (! date) return false;
	uint32_t newDate = atol(date);
	getDate(newDate);

	// no need to continue
	// if (altitude == NULL){
	// return true;
	//}
	return true;
}

/!\
