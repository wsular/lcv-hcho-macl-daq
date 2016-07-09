Mobile Air Chemistry Laboratory at Sunset Park
==============================================

Lewiston-Clarkston Valley Formaldehyde Study (2016)
---------------------------------------------------

Documentation for the Mobile Air Chemistry Laboratory (MACL) at Sunset Park.
The MACL contains gas-phase analyzers and meteorological sensors; measurements
include:

* At ~36ft (11m), by ultrasonic weather station ([WXT510; Vaisala][wxt510])
    * wind direction (avg/min/max)
    * wind speed (avg/min/max)
    * barometric pressure
    * air temperature
    * relative humidity
    * rain (accumluation/duration/intensity)
    * hail (accumulation/duration/intensity)
* At ~22ft (6.7m), via 1/2" I.D. PFA tubing sampling line by trace gas analyzers
    * carbon dioxide (CO2) ([LI-840A; LI-COR Biosciences][li840a])
    * water vapor (H2O) (also LI-840A)
    * carbon monoxide (CO) ([T300U; Teledyne API][t300u])
    * mono-oxides of nitrogen (NO/NO2/NOx) ([T200U; Teledyne API][t200u])
    * ozone (O3) ([T400; Teledyne API][t400])
    * sulfur dioxide (SO2) ([T100U; Teledyne API][t100u])
    * speciated volatile organic compounds (VOCs) ([*PTR-MS*; Ionicon][ptrms])
* At ~20ft (6.1m), by *some kind of NOy analyzer (some company)* **TODO FIXME**
    * reactive oxides of nitrogen (NOy)
* Within the trailer
    * sampling inlet mass flow ([4040 or 4043; TSI][4040]) **TODO FIXME**

Additionally, a ceilometer ([CL31; Vaisala][cl31]) deployed on the MACL roof
captures cloud height and coverage. Usage and data handling for the ceilometer
are not covered here.

> The PTR-MS and NOy analyer are listed for completeness; neither instrument
> is addressed here.

  [wxt510]: http://www.campbellsci.com/wxt510
  [li840a]: https://www.licor.com/env/products/gas_analysis/LI-840A/
  [t300u]: http://teledyne-api.com/products/T300U.asp
  [t200u]: http://teledyne-api.com/products/T200U_NOx.asp
  [t400]: http://teledyne-api.com/products/T400.asp
  [t100u]: http://teledyne-api.com/products/T100U.asp
  [ptrms]: http://www.ionicon.com/products/ptr-ms
  [4040]: http://tsi.com/Flowmeters/
  [cl31]: http://www.vaisala.com/en/maritime/products/ceilometer/Pages/CL31.aspx


### Acquisition

Data is aggregated into several locations. Where possible, measurements are
acquired redundantly to prevent data loss and provide the highest-quality data
signals. The higher-quality source is listed in the "primary DAQ" column in the
table below.

| Measurements            | Primary DAQ                | Secondary DAQ        |
|-------------------------|----------------------------|----------------------|
| meteorology (WXT510)    | [DAQFactory][daqf]         | *not applicable*     |
| sample mass flow        | DAQFactory                 | *not applicable*     |
| trace gases (Teledynes) | [APICOM][apicom]           | DAQFactory           |
| trace gases (LI-840A)   | [LI-840A Software][lisoft] | DAQFactory           |

  [daqf]: https://labjack.com/support/software/3rd-party-applications/daqfactory
  [apicom]: http://teledyne-api.com/software/apicom/index.asp
  [lisoft]: https://www.licor.com/env/products/gas_analysis/LI-840A/software.html

#### DAQFactory

> **TODO** verify DAQFactory model/version

A 19" rack-mounted PC running HMI/SCADA software (DAQFactory 5.87; Azeotech)
acquires data via analog voltage signals (for gas analyzers and flow meter) or
serial port (for weather station) and stores data to delimited text file. Raw 
measurements are aggregated on 60sec intervals into mean (average) values prior
to storage. A new file is created each day at midnight.

> **N.B.** All measurements are aggregated into mean values within the 1-minute
> logging interval, **including** wind direction avg/min/max and relative
> humidity from the WXT510. 

> Analog voltage signals are inherently susceptible to noise and bias. At low
> concentrations, the signal may be distorted significantly. To avoid this,
> it is recommended to use data in files created by APICOM or LI-840A Software
> in preference to DAQFactory data files.

#### APICOM

The Teledyne analyzers are configured to retain data in non-volatile onboard
memory:

* ~7 days of 1-min mean values (concentration(s) + stability indicator(s))
* ~31 days of 30-min mean values (*same params*)

Several instances of the manufacturer-provided communications utility
(APICOM 5.0.5; Teledyne API) run in the background on the rack PC, periodically
retrieving the latest records and appending them to cumulative delimited text 
files. One instance handles the retrieval for one analyzer. 

#### LI-840A Software

> **TODO** get version #

A single instance of the manufacturer-provided interface software (LI-840A
Software XXXXX; LI-COR Biosciences) runs in the background on the rack PC, performing
continuous acquisition of the serial data stream. The data is recorded without
aggregation (i.e. at 1Hz) into a cumulative delimited text file.

> In the interest of long-term reliability, **do not** engage the plotting
> features of LI-840A Software.


### Data Products

#### DAQFactory Files

* File name format: `Lewiston_RackYYMMDD.txt` where `YYMMDD` is date stamp
  using two-digit year
* Record interval: nominal 1-min (every 60sec/not aligned to minute) (anchors unknown)
* Aggregation: mean 1-min values of real-time measurements
    * **yes, naive mean applied even to WD & RH from WXT510**

| Column name  | Units        | Description                                  |
|--------------|--------------|----------------------------------------------|
| TheTime      | UTC-0800     | [unix epoch][epch] (Pacific Standard Time)   |
| LabjackU3_T  | degC         | temperature of DAQ hardware                  |
| LI840A_CO2   | ppm          | carbon dioxide mixing ratio                  |
| LI840A_H2O   | ppth         | water vapor mixing ratio                     |
| TAPI_CO      | ppb          | carbon monoxide mixing ratio                 |
| TAPI_NO      | ppb          | nitric oxide mixing ratio                    |
| TAPI_NO2     | ppb          | nitrogen dioxide mixing ratio                |
| TAPI_NOx     | ppb          | total mono-nitrogen oxides mixing ratio      |
| TAPI_O3      | ppb          | ozone mixing ratio                           |
| TAPI_SO2     | ppb          | sulfur dioxide mixing ratio                  |
| TSI_flow     | volts        | sampling inlet flow (unscaled)               |
| WXT_Dm       | m/s          | wind direction average                       |
| WXT_Dn       | m/s          | wind direction minimum                       |
| WXT_Dx       | m/s          | wind direction maximum                       |
| WXT_Hc       | hits/cm^2    | hail accumulation                            |
| WXT_Hd       | s            | hail duration                                |
| WXT_Hi       | hits/hr*cm^2 | hail intensity                               |
| WXT_Hp       | hits/hr*cm^2 | hail peak intensity                          |
| WXT_Pa       | mb           | air pressure                                 |
| WXT_Rc       | mm           | rain accumulation                            |
| WXT_Rd       | s            | rain duration                                |
| WXT_Ri       | mm/hr        | rain intensity                               |
| WXT_Rp       | mm/hr        | rain peak intensity                          |
| WXT_Sm       | m/s          | wind speed average                           |
| WXT_Sn       | m/s          | wind speed minimum                           |
| WXT_Sx       | m/s          | wind speed maximum                           |
| WXT_Ta       | degC         | air temperature                              |
| WXT_Ua       | %            | relative humidity                            |

Example: convert timestamp to [Excel serial date][excl] (assuming `TheTime` is
column A) ([ref][exclref]):
```excel
=(A1/86400)+DATE(1970,1,1)+(-8/24)
```

Example: import using [pandas][pd] in Python:
```python
import pandas as pd

df = pd.read_csv('/path/to/the/file.txt',
                 header=0, # headers 1st row
                 index_col=0, # timestamps 1st column
                 # interpret as seconds (since 1970)
                 date_parser=lambda x: pd.to_datetime(x, unit='s'),
                 parse_dates=True)
df.shift(-8, freq='H') # roll back UTC to PST (-0800)
```

  [excl]: https://support.office.com/en-us/article/Date-and-time-functions-reference-fd1b5961-c1ae-4677-be58-074152f97b81
  [exclref]: http://spreadsheetpage.com/index.php/tip/converting_unix_timestamps/
  [epch]: https://en.wikipedia.org/wiki/Unix_time
  [pd]: http://pandas.pydata.org

#### APICOM Files

* File name format: `XX_TTT_YYYYMMDD.csv` where
    * `XX` is the measured gases (`CO`, `NONO2`, `O3` or `SO2`)
    * `TTT` is either of `1min` or `30min`, representing 1-min or 30-min values
    * `YYYYMMDD` is the file creation date (not start of data in file)
* Record interval: 1 minute (closed left, label right) (timestamped at 1-second
  resolution, not 1-minute, thus label is of form `hh:mm:01`)
* Aggregation: mean 1-min values of real-time measurements

##### CO

| Column name      | Units    | Description                                  |
|------------------|----------|----------------------------------------------|
| Time Stamp       | UTC-0800 | timestamp in US date format (mm/dd) (PST)    |
| CONC1-AVG (PPM)  | ppm      | mean carbon monoxide mixing ratio            |
| STABIL-AVG (PPM) | ppm      |

##### NONO2

| Column name       | Units    | Description                                  |
|-------------------|----------|----------------------------------------------|
| Time Stamp        | UTC-0800 | timestamp in US date format (mm/dd) (PST)    |
| NOXCNC1-AVG (PPB) | ppb      | mean total mono-oxides of nitrogen mixing ratio |
| NOCNC1-AVG (PPB)  | ppb      | mean nitric acid mixing ratio                |
| NO2CNC1-AVG (PPB) | ppb      | mean nitrogen dioxide mixing ratio           |
| STABIL-AVG (PPM)  | ppm      |

##### O3

| Column name      | Units    | Description                                  |
|------------------|----------|----------------------------------------------|
| Time Stamp       | UTC-0800 | timestamp in US date format (mm/dd) (PST)    |
| CONC1-AVG (PPM)  | ppm      | mean ozone mixing ratio                      |
| STABIL-AVG (PPM) | ppm      |

> **TODO FIXME** why isn't o3 monitor recording stability parameter?

##### SO2

| Column name      | Units    | Description                                  |
|------------------|----------|----------------------------------------------|
| Time Stamp       | UTC-0800 | timestamp in US date format (mm/dd) (PST)    |
| CONC1-AVG (PPM)  | ppm      | mean sulfur dioxide mixing ratio             |
| STABIL-AVG (PPM) | ppm      |

Example: import using pandas in Python:
```python
import pandas as pd

df = pd.read_csv('/path/to/the/file.txt',
                 header=0, # headers 1st row
                 index_col=0, # timestamps 1st column
                 parse_dates=True, # US format (MM/DD/YYYY)
                 skipinitialspace=True) # spaces and commas
```

#### LI-840A Software Files

* File name format: `li840a_YYYYMMDD.txt` where `YYYYMMDD` is the file creation
  date (also typ. start of data in file)
* Record interval: 1 second
* Aggregation: none; represents real-time measurements

| Column name         | Units     | Description                              |
|---------------------|-----------|------------------------------------------|
| Date(Y-M-D)         | *n/a*     | date portion of timestamp (PST)          |
| Time(H:M:S)         | UTC-0800  | time portion of timestamp (PST)          |
| CO2(ppm)            | ppm       | carbon dioxide mixing ratio              |
| H2O(ppt)            | ppth      | water vapor mixing ratio                 |
| H2O(C)              | degC      | water vapor dewpoint                     |
| Cell_Temperature(C) | degC      | temperature in measurement cell          |
| Cell_Pressure(kPa)  | kPa       | pressure in measurement cell             |


Example: import using pandas in Python:
```python
import pandas as pd

df = pd.read_table('/path/to/the/file.txt', # table for tab-separated
                   header=1, # headers 2nd row
                   index_col=0 # dates 1st col, times 2nd col
                   parse_dates={'timestamp': [0,1]}) # combine cols
```


### Initial Device Setup

Step-by-step instructions are available in the relevant user manuals. See
[References](#markdown-header-references) below.

#### Weather Station (WXT510)

> **TODO**


#### Teledyne analyzers

> **TODO**


#### Router

The router contains DHCP reservations and port forwarding rules which permit
devices to talk to one another.

> **TODO**

#### Broadband Modem (Raven X)

> **TODO**

Review this:

```
Login to the device's web interface (typ. <http://192.168.13.31:9191> from the
LAN side) and set the following configuration (on factory defaults):

* LAN > Ethernet
    * Host Public Mode: Ethernet Uses Public IP
* LAN > Global DNS (optional, recommended)
    * Alternate Primary DNS: *<WSU primary nameserver IP address>*
    * Alternate Secondary DNS: *<WSU secondary nameserver IP address>*
* Services > ACEManager (optional, strongly recommended for emergency remote
  administration)
    * OTA ACEmanager Access: SSL Only
* Services > Telnet/SSH
    * AT Server Mode: SSH
    * [Make SSH Keys] button: Press if SSH is not initalized 
* Services > Time (SNTP)
    * Enable time update: Enabled
    * SNTP Server Address: *[pick one](http://www.pool.ntp.org/en/)*

Apply and reboot.
```


#### Rack PC

The rack PC runs DAQFactory, APICOM, LI-840A Software and other important
utilities.

list desktops & show pictures

> **TODO**





### References

* Campbell Scientific, Inc. *WXT510 Weather Transmitter Instruction Manual.*
  Revision 2007 Apr. 
  Online: <https://s.campbellsci.com/documents/us/manuals/wxt510.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T100 UV Flourescence SO2
  Analyzer Operation Manual.* Revision 2015 June 28.
  Online: <http://teledyne-api.com/manuals/06807B_T100.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T100U Trace Level Sulfur
  Dioxide Analyzer (Addendum to T100 Operation Manual, PN 06807).* 
  Revision 2011 Aug 11.
  Online: <http://teledyne-api.com/manuals/06840B_T100U_Addendum.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T200 NO/NO2/NOx Analyzer
  Operation Manual.* Revision 2015 Aug 4.
  Online: <http://teledyne-api.com/manuals/06858_T200.pdf>

* Teledyne Advanced Pollution Instrumentation. *Ultra Sensitivity Model T200U
  NO/NO2/NOx (Addendum to T200 Manual, PN 06858).* Revision 2013 Feb 19.
  Online: <http://teledyne-api.com/manuals/06861C_T200UNOx_%20Addendum.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T300/T300M Carbon
  Monoxide Analyzer Operation Manual.* Revision 2012 Feb 14.
  Online: <http://teledyne-api.com/manuals/06864B_T300_M.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T300U CO Analyzer with
  Auto-Reference (Addendum to T300/T300M Operation Manual, PN 06864).*
  Revision 2012 Jan 30.
  Online: <http://teledyne-api.com/manuals/06867B_T300U_Addendum.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T400 Photometric Ozone
  Analyzer Operation Manual.* Revision 2014 Sep 10.
  Online: <http://teledyne-api.com/manuals/06870E_T400.pdf>

* Teledyne Advanced Pollution Instrumentation. *Model T700 Dynamic Dilution
  Calibrator Operation Manual.* Revision 2015 Jun 10.
  Online: <http://teledyne-api.com/manuals/06873C_T700.pdf>

* Vaisala Oyj. *Vaisala Weather Transmitter WXT510 User's Guide.* 
  Revision 2006.
  Online: <http://www.vaisala.com/Vaisala%20Documents/User%20Guides%20and%20Quick%20Ref%20Guides/WXT510_User_Guide_in_English.pdf>

