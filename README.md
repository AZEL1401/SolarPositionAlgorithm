# SolarPositionAlgorithm
**NREL Solar Position Algorithm implemented in C#**. 

This algorithm calculates the solar zenith and azimuth angles in the period from the year -2000 to 6000, with uncertainties of 0.0003 degrees based on the date, time, and location on Earth. (Reference: Reda, I.; Andreas, A., Solar Position Algorithm for Solar Radiation Applications, Solar Energy. Vol. 76(5), 2004; pp. 577-589). The software has not been tested on a variety of platforms and is not guaranteed to work on yours. It is provided here as a convenience.
Further information on this algorithm is available in the following NREL technical report: 

[Reda, I.; Andreas, A. (2003). Solar Position Algorithm for Solar Radiation Applications. 55 pp.; NREL Report No. TP-560-34302, Revised January 2008.](http://www.nrel.gov/docs/fy08osti/34302.pdf)

Original code in C/C++ and licence can be found [here](http://www.nrel.gov/midc/spa/).

Example of use:
```cs
using System;

using SPACalculator;
using SPACalculator.Enums;
using SPACalculator.Models;
using SPACalculator.Models.IntermediateModels;

namespace SPATestConsole
{
	class Program
	{
		static void Main()
		{
			var enironment = new EnviromentModel
			{
				Longitude = -105.1786,
				Latitude = 39.742476,
				Elevation = 1830.14,
				Pressure = 820,
				Temperature = 11,
				Slope = 30,
				AzmRotation = -10,
				AtmosRefract = 0.5667,
			};
			var time = new TimeModel
			{
				Year = 2003,
				Month = 10,
				Day = 17,
				Hour = 12,
				Minute = 30,
				Second = 30,
				Timezone = -7.0,
			};
			var timeDeltas = new TimeDeltasModel
			{
				DeltaUt1 = 0,
				DeltaT = 67,
			};

			var data = new DataModel
			{
				Enviroment = enironment,
				Time = time,
				TimeDeltas = timeDeltas,
				MidOut = new IntermediateOutputModel(),
				Output = new OutputModel(),
				Mode = CalculationMode.All
			};

			var result = SPACalculator.SPACalculator.SPACalculate(ref data);

			// Check for SPA errors
			if (result == 0)
			{
				Console.Write("Julian Day:    {0}\n", data.MidOut.JulianTime.JDay);
				Console.Write("HeliocentericLongitude:             {0} degrees\n", data.MidOut.EarthMidOut.HeliocentericLongitude);
				Console.Write("HeliocentericLatitude:             {0} degrees\n", data.MidOut.EarthMidOut.HeliocentericLatitude);
				Console.Write("RadiusVector:             {0} AU\n", data.MidOut.EarthMidOut.RadiusVector);
				Console.Write("H:             {0} degrees\n", data.MidOut.H);
				Console.Write("Delta Psi:     {0} degrees\n", data.MidOut.DelPsi);
				Console.Write("Delta Epsilon: {0} degrees\n", data.MidOut.DelEpsilon);
				Console.Write("Epsilon:       {0} degrees\n", data.MidOut.Epsilon);
				Console.Write("Zenith:        {0} degrees\n", data.Output.Zenith);
				Console.Write("Azimuth:       {0} degrees\n", data.Output.Azimuth);
				Console.Write("Incidence:     {0} degrees\n", data.Output.Incidence);

				var min = 60.0 * (data.Output.Sunrise - (int)data.Output.Sunrise);
				var sec = 60.0 * (min - (int)min);
				Console.Write("Sunrise:       {0}:{1}:{2} Local Time\n", (int)data.Output.Sunrise, (int)min, (int)sec);

				min = 60.0 * (data.Output.Sunset - (int)data.Output.Sunset);
				sec = 60.0 * (min - (int)min);
				Console.Write("Sunset:        {0}:{1}:{2} Local Time\n", (int)data.Output.Sunset, (int)min, (int)sec);
			}
			else
			{
				Console.WriteLine("SPA Error Code: {0}", result);
			}

			Console.ReadKey();
		}
	}
}
```
This code should output the following result:
```
Julian Day:    2452930.31284722
L:             24.0182616916794 degrees
B:             -0.000101121924800342 degrees
R:             0.996542297353971 AU
H:             11.1059020139134 degrees
Delta Psi:     -0.00399840430333278 degrees
Delta Epsilon: 0.00166656817724969 degrees
Epsilon:       23.4404645196175 degrees
Zenith:        50.1116220240297 degrees
Azimuth:       194.340240510192 degrees
Incidence:     25.1870002003532 degrees
Sunrise:       6:12:43 Local Time
Sunset:        17:20:19 Local Time
```

But due to bugs appeared in refactoring (99.9% because of breaking data struct to some happy classes!) it's output is (+3:30GMT):
```
Julian Day:    2452931.5
HeliocentericLongitude:             24.203218344625306 degrees
HeliocentericLatitude:             -0.00010445945805785599 degrees
RadiusVector:             0.9964909193054843 AU
H:             11.105902013913436 degrees
Delta Psi:     -0.003995186150812188 degrees
Delta Epsilon: 0.0016671492073479675 degrees
Epsilon:       23.440465034298906 degrees
Zenith:        50.11162202402972 degrees
Azimuth:       194.34024051019162 degrees
Incidence:     25.18700020035315 degrees
Sunrise:       13:12:43 Local Time
Sunset:        0:20:19 Local Time
```
