# Introduction #

This repository contains an example project that demonstrates how to read NetCDF data from S3 or a local filesystem into a Spark/Scala program using [NetCDF Java](http://www.unidata.ucar.edu/software/thredds/current/netcdf-java/) and manipulate the data using [GeoTrellis](https://geotrellis.io/).
The ability to easily and efficiently read NetCDF data into a GeoTrellis program opens the possibility for those who are familiar with GeoTrellis and its related and surrounding tools to branch into climate research, and also makes it possible for climate researchers to take advantage of the many benefits that GeoTrellis can provide.
Because GeoTrellis is a raster-oriented library, the approach that is demonstrated in this repository is to use the NetCDF library to load and query datasets and present the results as Java arrays which can be readily turned into GeoTrellis tiles.
Once the data have been transformed into GeoTrellis tiles, they can be masked, summarized, and/or manipulated like any other GeoTrellis raster data.
The results of that are shown below.

In the last section there is a brief discussion of ideas for improving the S3 Reader.

# Compiling And Running #

## Dependencies ##

This code relies on two main dependencies to do most of the work: NetCDF Java and GeoTrellis.

### NetCDF Java ###

Because the S3 and HDFS reading capability are not present in mainline NetCDF Java, you must compile and locally-publish a [particular feature branch](https://github.com/Unidata/thredds/tree/feature/s3+hdfs) that was recently contributed by the present author and will hopefully make its way into the mainline at some point.
To compile and locally-publish the feature branch, try something like the following:

```bash
git clone 'git@github.com:Unidata/thredds.git'
cd thredds/
git fetch origin 'feature/s3+hdfs:feature/s3+hdfs'
git checkout 'feature/s3+hdfs'
./gradlew assemble
./gradlew publishToMavenLocal
```

### GeoTrellis ###

This code requires a [1.2.0-SNAPSHOT](https://github.com/locationtech/geotrellis) or later version of GeoTrellis.
That is due to the fact that recently-added tile transformation functions are used in this code which are not present in earlier version of GeoTrellis.
To compile and locally-publish GeoTrellis, try this:

```bash
git clone 'git@github.com:locationtech/geotrellis.git'
cd geotrellis/
./scripts/publish-local.sh
```

## Compile ##

With the dependencies in place, compiling the code in this repository is straightforward:

```bash
sbt "project gddp" assembly
```

## Run ##

To run the code from the root directory of the project, try this:

```bash
$SPARK_HOME/bin/spark-submit --master 'local[*]' \
   gddp/target/scala-2.11/gddp-assembly-0.22.7.jar \
   /tmp/gddp.nc /tmp/boundary.json '32.856388889,-90.4075'
```

where the first argument (after the name of the jar file) is the name of a GDDP NetCDF file, the second argument is the name of a file that contains a polygon in GeoJSON format, and the third argument is a latitude-longitude pair.

The program will produce several outputs:
   - A png of the first tile (the tile for day 0) in the given GDDP dataset, that will appear in `/tmp/gddp.png`.
   - A png of the first tile clipped to the extent of the GeoJSON polygon that was given, that will appear in `/tmp/gddp1.png`.
   - A png of the first tile clipped to the extent of the GeoJSON polygon and masked against that polygon, that will appear in `/tmp/gddp2.png`.
   - Time series of the minimum, mean, and maximum temperatures in the area enclosed by the polygon, as well as temperatures at the given point.

## Example 1 (Local File) ##

If you download the file `s3://nasanex/NEX-GDDP/BCSD/rcp85/day/atmos/tasmin/r1i1p1/v1.0/tasmin_day_BCSD_rcp85_r1i1p1_inmcm4_2099.nc` and put it into a local file called `/tmp/tasmin_day_BCSD_rcp85_r1i1p1_inmcm4_2099.nc`,
then type:

```bash
$SPARK_HOME/bin/spark-submit --master 'local[*]' \
   gddp/target/scala-2.11/gddp-assembly-0.22.7.jar \
   /tmp/tasmin_day_BCSD_rcp85_r1i1p1_inmcm4_2099.nc \
   ./geojson//CA.geo.json \
   '33.897,-118.225'
```

You will get the following results.

The whole first tile:
![gddp](https://cloud.githubusercontent.com/assets/11281373/26764732/b5f2813c-493a-11e7-8ad7-746527b5988f.png)

The first tile clipped to the extent of the polygon (the boundary of California):
![gddp1](https://cloud.githubusercontent.com/assets/11281373/26764734/b5f59868-493a-11e7-8364-878a05882717.png)

The first tile clipped and masked:
![gddp2](https://cloud.githubusercontent.com/assets/11281373/26764733/b5f2f0cc-493a-11e7-840d-bfb5239aabf1.png)

The time series:
```
MINS: List(265.72344970703125, 270.7049865722656, 271.9108581542969, 268.8466491699219, 267.29541015625, 269.53277587890625, 273.539306640625, 271.26629638671875, 270.24981689453125, 269.14605712890625, 270.0053405761719, 266.7234802246094, 268.25787353515625, 264.21490478515625, 268.3309631347656, 267.36083984375, 271.8747863769531, 270.132080078125, 269.6143493652344, 273.75494384765625, 274.6582336425781, 274.4009704589844, 266.7240905761719, 265.91009521484375, 266.1998291015625, 270.10626220703125, 269.57373046875, 269.2007751464844, 269.190185546875, 269.040771484375, 269.97900390625, 271.5916748046875, 275.49566650390625, 278.1347961425781, 276.7024841308594, 278.17620849609375, 276.9932861328125, 276.43475341796875, 275.04266357421875, 271.9781188964844, 269.7855529785156, 267.8652648925781, 271.53692626953125, 267.67913818359375, 269.4219055175781, 274.9229736328125, 273.8660888671875, 273.7021484375, 274.4654541015625, 272.4128112792969, 266.5999755859375, 262.75311279296875, 259.9469299316406, 261.53631591796875, 262.03448486328125, 266.5677490234375, 263.3155517578125, 260.87872314453125, 260.8282165527344, 255.34481811523438, 256.80633544921875, 258.6957092285156, 262.7079162597656, 265.5004577636719, 265.5907897949219, 256.5030517578125, 255.06959533691406, 256.94891357421875, 254.15721130371094, 257.4886169433594, 257.2295227050781, 259.6545104980469, 257.2198486328125, 255.9344940185547, 257.6353759765625, 256.301025390625, 255.65870666503906, 261.52716064453125, 262.96917724609375, 260.8644714355469, 262.8670654296875, 260.8768615722656, 255.18325805664062, 256.2804260253906, 252.03968811035156, 253.2799530029297, 256.68560791015625, 258.6305236816406, 256.03350830078125, 258.5773620605469, 263.7726745605469, 263.10357666015625, 264.526611328125, 266.41925048828125, 265.87811279296875, 266.40673828125, 265.9853820800781, 269.7365417480469, 268.96966552734375, 264.44061279296875, 262.60577392578125, 264.8210754394531, 264.77105712890625, 268.9519348144531, 266.815673828125, 262.1197509765625, 260.7754821777344, 263.9394836425781, 272.97900390625, 270.1040954589844, 270.19879150390625, 269.95263671875, 266.11090087890625, 263.71160888671875, 263.75811767578125, 262.91729736328125, 262.2646179199219, 262.31103515625, 268.98858642578125, 271.04364013671875, 271.6297607421875, 273.12030029296875, 272.35986328125, 272.53564453125, 274.25250244140625, 271.03765869140625, 271.1427917480469, 271.0518798828125, 271.93536376953125, 272.779052734375, 274.531005859375, 276.93115234375, 280.10986328125, 278.2539978027344, 277.3471374511719, 276.2685546875, 272.5466613769531, 275.9110107421875, 274.5663757324219, 268.27874755859375, 270.32501220703125, 270.875244140625, 273.2278137207031, 274.0507507324219, 274.2288818359375, 276.0082092285156, 274.6853942871094, 273.7604064941406, 275.3476257324219, 276.3729553222656, 272.12371826171875, 272.8186950683594, 272.6412658691406, 273.9280090332031, 273.6745300292969, 274.70770263671875, 274.7071228027344, 272.7940979003906, 272.66583251953125, 273.7194519042969, 272.72723388671875, 275.1534729003906, 277.5609130859375, 278.9190673828125, 280.24560546875, 279.1988220214844, 278.0628356933594, 274.50604248046875, 274.6471862792969, 274.54998779296875, 273.7932434082031, 274.79925537109375, 276.05694580078125, 276.258056640625, 275.9925537109375, 273.1938781738281, 272.7206115722656, 273.4480285644531, 273.5872802734375, 274.2318115234375, 275.94659423828125, 275.96844482421875, 275.3699951171875, 272.7214660644531, 273.64385986328125, 274.70159912109375, 278.6965637207031, 277.56396484375, 275.9771423339844, 277.2000732421875, 275.929443359375, 278.0752868652344, 277.711669921875, 278.4751892089844, 279.3154602050781, 280.8727111816406, 278.20196533203125, 274.80914306640625, 274.9639892578125, 276.8976135253906, 280.6986083984375, 280.2890625, 279.9993591308594, 279.5530700683594, 277.97186279296875, 275.54345703125, 275.7853698730469, 274.1766052246094, 273.53936767578125, 275.5015869140625, 277.3805847167969, 273.81890869140625, 276.416015625, 277.745849609375, 279.79254150390625, 279.461181640625, 280.66162109375, 278.6689453125, 281.3240661621094, 278.6739807128906, 276.6484069824219, 276.95361328125, 278.0138244628906, 278.9786682128906, 278.3924255371094, 278.2974548339844, 275.76007080078125, 280.0055236816406, 280.237548828125, 278.50640869140625, 276.7699890136719, 279.46209716796875, 279.3669128417969, 280.8971252441406, 277.4842224121094, 279.0671691894531, 280.1692810058594, 279.00323486328125, 276.1126403808594, 276.9248046875, 276.15643310546875, 275.40338134765625, 274.99505615234375, 272.9776306152344, 274.00897216796875, 275.259765625, 276.51361083984375, 278.3191833496094, 278.85101318359375, 278.8699035644531, 278.47613525390625, 276.3825988769531, 276.8254089355469, 279.3219909667969, 274.0939636230469, 272.2311706542969, 270.87017822265625, 275.761962890625, 273.4514465332031, 267.9302062988281, 269.4923400878906, 273.3327941894531, 275.2063293457031, 273.44757080078125, 273.2978820800781, 275.6490173339844, 275.56097412109375, 274.6640930175781, 271.8046875, 270.23388671875, 270.6744079589844, 268.8171691894531, 270.94256591796875, 274.8165283203125, 271.312255859375, 270.12298583984375, 271.6394348144531, 270.3510437011719, 266.8189697265625, 267.5841369628906, 268.6550598144531, 271.1710205078125, 275.7001037597656, 275.77691650390625, 276.4483642578125, 274.5020751953125, 271.7063903808594, 270.65765380859375, 271.8366394042969, 269.76153564453125, 264.4934997558594, 264.6595764160156, 266.8653869628906, 266.2351989746094, 263.81280517578125, 262.87689208984375, 265.19329833984375, 261.34588623046875, 262.4715270996094, 261.6071472167969, 258.37603759765625, 257.2313232421875, 259.1628112792969, 259.34539794921875, 260.75048828125, 262.2595520019531, 261.0798034667969, 261.3926086425781, 259.43267822265625, 260.2183532714844, 259.36334228515625, 262.28509521484375, 261.7064208984375, 263.4981384277344, 262.3600158691406, 256.65234375, 258.5657653808594, 256.65411376953125, 256.339599609375, 258.7945556640625, 257.5079650878906, 255.3631591796875, 256.7193603515625, 258.3073425292969, 259.9819641113281, 259.6513671875, 256.0986633300781, 257.74676513671875, 257.8719177246094, 251.92665100097656, 254.39918518066406, 265.2249755859375, 262.8497009277344, 263.60906982421875, 261.8241882324219, 262.3623962402344, 266.6568603515625, 264.5255126953125, 265.0112609863281, 268.7845153808594, 268.2677917480469, 263.84381103515625, 263.7473449707031, 262.81256103515625, 261.6624450683594, 258.84381103515625, 258.5262756347656, 258.4081726074219, 256.66802978515625, 254.211181640625, 261.0124816894531, 257.706298828125, 266.6851501464844, 269.40765380859375, 263.1731262207031, 268.1440734863281, 266.4936828613281, 263.03619384765625, 263.70782470703125, 261.6356506347656, 262.10076904296875, 262.00543212890625, 261.4680480957031, 261.05462646484375, 261.35028076171875)
MEANS: List(283.77571613067306, 284.76922671094917, 285.15654003069403, 282.66115564383267, 281.1060727342586, 283.52409492413085, 284.81623587913913, 283.83160545929, 283.41879786393326, 284.7069251889088, 284.2113523298511, 282.3007443196372, 280.79229186011145, 278.9953607823561, 282.1755982809735, 283.1732069945016, 284.97437555850354, 284.7299268924355, 283.83099397070896, 287.7258025167951, 287.34076227907275, 288.06341748301566, 284.8482686481959, 279.60369468268266, 281.36208519743735, 285.4235608801579, 284.22331262807376, 282.5761046999435, 283.1710898659446, 282.9379515328102, 283.85852805761573, 285.66671502785607, 288.463337063967, 291.37037856532635, 289.9845485090321, 290.6477128553319, 289.3550972178156, 288.9684861207328, 288.4874083835926, 288.5031906105189, 287.45118092566776, 282.4096163935882, 284.6594907757778, 281.8096353499616, 284.7789681401942, 287.89307218230425, 287.8819387239777, 287.3338589391126, 288.80635004853707, 285.0984527997217, 280.24172073114113, 278.93569746173796, 278.35619166639987, 279.09997003728694, 278.66642254120194, 281.0432968480754, 278.27806443784345, 277.47964686750475, 277.6001087535511, 277.1723902740706, 277.32206714704034, 277.9652301089064, 280.3953839229578, 282.0949051653753, 281.0874155331653, 275.39491078679504, 273.34233843693966, 275.8456403160948, 277.4585253833481, 276.4875898517546, 277.7773959164115, 277.2104351197199, 273.851526661175, 274.03217118969616, 275.9378816743899, 275.23548623097815, 275.0583323039525, 277.7149228864917, 278.3819686503062, 278.52066076423654, 278.98825241520933, 277.8753204573107, 272.0425172626706, 271.6145010889909, 270.09722386458236, 272.7673334386061, 275.14160551932457, 275.8453778922114, 274.74034630310695, 277.1938211409772, 279.7478456681958, 282.54110822379147, 282.96262751890544, 281.43555883643523, 281.85556953532506, 282.60673889069193, 282.83242320303765, 284.9937772793493, 283.7966819342487, 281.5170722427027, 279.93404996732664, 281.92681466342793, 282.10161771362124, 283.73828416076753, 280.86077844474784, 278.800985322091, 278.3720347920402, 281.75883913679735, 287.03788004617934, 286.53023364185043, 285.1750343197682, 283.64234799244366, 280.4606568383389, 279.3301853685905, 279.76347953694057, 278.2664527949917, 277.5062383205454, 279.2601165089273, 283.1139669631526, 283.8085896567332, 285.0492800216561, 287.1741580423051, 287.13681053013806, 286.39338695452216, 287.5329734927318, 287.76545180933306, 287.58570086369747, 286.67395842732685, 286.914031436653, 286.4665235812191, 287.4879073073363, 289.5319124724933, 291.6430555818333, 291.5622858311842, 290.07319970848664, 288.9463449128516, 288.60800348273335, 289.3925927243183, 288.6474250531943, 284.64525243813165, 284.7026566847959, 285.1740437037188, 286.17186745019677, 286.88866678115687, 286.9161166832095, 288.3749979533666, 287.7297276413032, 287.944851234311, 288.32661394781576, 288.33776409757473, 288.1740883658078, 286.3675726309263, 287.0650417822484, 288.056639033174, 287.027586298799, 288.25197704787047, 288.32596248166215, 287.32826621770505, 286.48598797452786, 287.5485517840094, 288.99120147750557, 289.5618579938408, 290.49488500831023, 290.9413735493462, 291.08989546160404, 290.7651364003848, 289.8300941342213, 288.980740952243, 287.6888631033293, 287.3969234915675, 287.4435166019263, 288.54328820673436, 288.47044038275135, 289.5354124430217, 290.2198662793405, 287.87446432781644, 287.44704663522555, 287.82799769940215, 288.6234083104595, 288.47402067227085, 288.7478894661507, 289.75588966517444, 289.93168942583657, 288.93395841459227, 289.2094450949201, 289.6843013848704, 290.4909247726691, 290.0956045682313, 289.7215976857215, 290.4181095583784, 291.1568118236104, 292.301364858886, 292.2102616106878, 292.0515279528281, 292.24480050509095, 293.15895041907777, 291.90439429475015, 290.27578289640286, 290.83880292321106, 290.84308302491326, 292.76461105232977, 292.2709347857093, 292.07154634705955, 291.55899234322607, 290.83438828947294, 289.0751393711869, 288.949053518463, 289.16825027579523, 289.4135250540675, 290.448493559563, 291.94809355643395, 290.4861594369266, 290.45982849757826, 291.14827499730757, 292.66277403056887, 292.35328326388725, 293.1217066030033, 292.2049776125594, 292.48165329593485, 290.3985284614847, 289.2087643846492, 289.52561680073177, 290.2295462186219, 291.6021673938734, 291.99554679859233, 291.2435071692204, 290.66815112777687, 291.0335676076693, 292.5464038649899, 293.06389712292463, 291.91180174325865, 292.8169959078247, 292.905257200875, 293.475597762493, 292.01166931243307, 292.5222373086898, 293.1354888142843, 292.61791737495344, 291.8558335510345, 291.44341213266114, 291.50148171710543, 291.2500800915873, 290.751275780304, 289.7321008719204, 290.2366736421997, 290.84360659921936, 291.68359538730675, 291.88322987677265, 291.8480738541764, 291.94792250336195, 291.0041060469012, 289.98688467284427, 290.20672843921733, 291.69885162944763, 289.40405851042925, 288.3573183608304, 287.50280711689936, 288.02405940367106, 286.6380687548755, 284.17464541394025, 285.5800996828719, 287.5547887401325, 288.4551445723646, 288.10239161393326, 288.54399615996994, 289.9859152053401, 289.1047746683908, 288.95706790747835, 288.02407886942876, 286.75826680713544, 286.3923031939124, 287.60170039761084, 289.6218346767738, 288.6451147679245, 287.8582653153671, 287.77894362607407, 286.4028461302801, 284.47656854893876, 282.3912716451949, 282.246531365703, 282.69402066499396, 285.6984023706746, 289.8670504210425, 289.7791798530499, 289.9249419483804, 288.4617094446401, 286.68821322900175, 287.07215619833386, 286.60555170687405, 286.2480567898018, 283.0406681066476, 283.51785996916044, 283.69759199480717, 281.457806924061, 279.406988425333, 278.5447474108723, 280.9308193337544, 277.84194800750686, 278.88552956502946, 279.12745527787644, 275.48788224744726, 274.14177479864765, 276.58217979010453, 276.98380080562765, 277.22925500542857, 277.4048734343709, 277.0626775568182, 276.71180299852716, 276.14647004856613, 275.6735774351481, 276.17638568906597, 278.5290359519811, 280.940539841858, 280.610570461313, 278.9065588597214, 276.2451017240476, 275.4896203394974, 275.3950389970077, 276.0499981170973, 276.94889447230696, 276.41489570720006, 275.780722878196, 274.92195622860584, 276.06169139788153, 277.25420210222904, 276.8786937184909, 274.1552701174058, 275.84349940110957, 278.0362167699504, 274.10797758145054, 276.14408633463785, 280.3741167185026, 278.8387878690853, 278.9380992997421, 278.8556907869842, 277.46493571206105, 278.7399726333277, 279.3815560944922, 280.0553017174256, 284.10114994418603, 283.06125237227553, 279.7650192874732, 279.90299871817075, 277.89658952647636, 277.7670835219268, 275.42570288668094, 276.2881048574888, 274.84106049630043, 275.732131662383, 274.01863464264505, 275.1687376929111, 276.96436583835924, 281.82687764516, 281.9555209709884, 279.04325782559846, 282.0491397135659, 280.95353155235597, 279.1602723168545, 277.24807802930854, 275.5143993849548, 275.66596873435464, 276.967277742889, 278.5729214118241, 276.7489155571791, 275.9928254583731)
MAXS: List(293.6987609863281, 293.4059753417969, 296.8443298339844, 292.9392395019531, 290.55487060546875, 289.97698974609375, 292.98651123046875, 291.2919921875, 291.287841796875, 291.5873107910156, 291.3080749511719, 288.8144836425781, 289.9812927246094, 286.6343078613281, 291.3164367675781, 290.52447509765625, 293.4313049316406, 290.6448059082031, 290.6263427734375, 298.19500732421875, 297.1639099121094, 296.6749572753906, 295.38775634765625, 289.2779541015625, 288.9710388183594, 293.1408386230469, 291.4801025390625, 291.9001159667969, 291.09796142578125, 291.2861633300781, 292.61236572265625, 293.85247802734375, 297.2470397949219, 299.53265380859375, 298.83074951171875, 300.11029052734375, 297.42047119140625, 298.8979797363281, 300.9543762207031, 299.7547912597656, 297.6268615722656, 290.56048583984375, 293.60382080078125, 290.6556091308594, 292.0706787109375, 295.83428955078125, 297.7505187988281, 298.833740234375, 298.2948303222656, 293.77532958984375, 288.6759033203125, 288.004150390625, 287.3899841308594, 286.27813720703125, 285.9300537109375, 288.50201416015625, 288.536376953125, 287.9044189453125, 286.86138916015625, 287.55877685546875, 288.41668701171875, 288.274658203125, 288.9968566894531, 288.5765075683594, 288.6041259765625, 288.08795166015625, 283.861328125, 285.78741455078125, 288.22088623046875, 285.3238830566406, 285.9909973144531, 286.4778137207031, 282.5812072753906, 282.8723449707031, 284.80511474609375, 283.7980041503906, 284.8526306152344, 284.5890808105469, 286.4769592285156, 287.531494140625, 287.3345031738281, 288.3788146972656, 284.3114929199219, 282.77490234375, 280.3612060546875, 282.6458740234375, 286.1882019042969, 287.99945068359375, 287.9600830078125, 284.4983825683594, 286.1615295410156, 290.1078186035156, 289.7294921875, 288.90997314453125, 290.6408386230469, 288.96173095703125, 291.0360412597656, 293.4213562011719, 294.3285827636719, 290.10528564453125, 288.3388366699219, 289.59136962890625, 290.2420654296875, 294.2307434082031, 291.4642639160156, 290.4881896972656, 288.2763977050781, 289.2904052734375, 294.63958740234375, 296.0103759765625, 296.4288024902344, 296.6001892089844, 293.8957214355469, 289.0975341796875, 289.1241455078125, 287.43408203125, 287.4101867675781, 288.9337158203125, 291.1912536621094, 295.0312805175781, 296.50927734375, 296.6528625488281, 296.87774658203125, 295.9515075683594, 296.9661560058594, 298.05218505859375, 298.4864196777344, 297.4093322753906, 296.7171325683594, 297.00244140625, 298.5117492675781, 300.64996337890625, 303.98974609375, 305.5233154296875, 302.2680969238281, 300.3421630859375, 300.147216796875, 299.3858337402344, 298.975341796875, 298.3555603027344, 298.12945556640625, 295.5409240722656, 298.362060546875, 298.69873046875, 298.8436279296875, 299.49151611328125, 299.1773376464844, 298.8034973144531, 300.2510986328125, 302.9501037597656, 303.0295104980469, 300.23974609375, 301.01995849609375, 300.532958984375, 298.37957763671875, 298.3681335449219, 299.12261962890625, 298.97796630859375, 297.5640563964844, 296.7811279296875, 296.3105773925781, 298.4842529296875, 301.2179870605469, 302.95623779296875, 304.2931213378906, 303.46337890625, 302.12286376953125, 299.2693786621094, 297.8948974609375, 298.3386535644531, 298.37615966796875, 298.6490478515625, 299.42791748046875, 299.9925537109375, 300.1806335449219, 297.83941650390625, 301.1888122558594, 301.9557189941406, 301.98028564453125, 301.4869079589844, 301.7799377441406, 303.8699645996094, 303.9251403808594, 305.6645812988281, 303.5476989746094, 303.40045166015625, 303.1889953613281, 301.6794738769531, 302.43267822265625, 300.82427978515625, 303.5585632324219, 303.3559875488281, 304.5736083984375, 304.94110107421875, 304.6776123046875, 306.7261962890625, 306.4850158691406, 305.22467041015625, 302.635986328125, 303.4544677734375, 304.6860046386719, 307.1360778808594, 304.5229797363281, 304.4716491699219, 302.1427307128906, 299.897705078125, 300.5875549316406, 299.31927490234375, 300.0832824707031, 302.50537109375, 302.70220947265625, 302.9096374511719, 302.507568359375, 303.4606628417969, 304.9081726074219, 305.70294189453125, 305.8072204589844, 305.8340148925781, 304.9718322753906, 304.1661682128906, 303.1705322265625, 302.2763366699219, 301.3083801269531, 303.3945007324219, 306.4698181152344, 303.7028503417969, 303.2659912109375, 305.6767272949219, 306.3843688964844, 306.6290283203125, 306.73248291015625, 307.6687927246094, 307.8703918457031, 305.0333251953125, 302.3546142578125, 301.52117919921875, 303.19610595703125, 303.0143127441406, 305.1505126953125, 304.54803466796875, 302.78704833984375, 301.8176574707031, 301.49273681640625, 302.27069091796875, 302.32855224609375, 301.5742492675781, 302.73565673828125, 303.59149169921875, 303.9085998535156, 304.11077880859375, 302.9640808105469, 302.9539489746094, 303.332763671875, 304.66986083984375, 301.4873046875, 298.9671325683594, 295.88665771484375, 299.3241882324219, 297.0010070800781, 297.4910888671875, 298.5479431152344, 299.916748046875, 300.3934020996094, 298.44720458984375, 300.4591064453125, 302.34765625, 303.5223083496094, 301.9506530761719, 301.229736328125, 298.3123474121094, 295.32696533203125, 299.5208435058594, 300.25048828125, 295.2781677246094, 297.1023864746094, 296.82659912109375, 295.67791748046875, 293.1402587890625, 292.29345703125, 291.0386047363281, 291.42779541015625, 294.2127990722656, 296.0615539550781, 297.8889465332031, 297.8788146972656, 296.7337341308594, 295.1321105957031, 294.20166015625, 293.3591613769531, 295.0370178222656, 292.84869384765625, 291.5419921875, 291.300048828125, 291.5111999511719, 291.6973571777344, 291.12969970703125, 292.8562316894531, 292.5498046875, 289.99993896484375, 290.8387756347656, 289.608642578125, 287.168212890625, 285.66455078125, 286.40972900390625, 287.8975524902344, 288.9295654296875, 288.17242431640625, 289.15863037109375, 288.8133850097656, 288.5594482421875, 286.51739501953125, 289.7039489746094, 287.6817932128906, 287.7493896484375, 287.925048828125, 285.28961181640625, 285.6154479980469, 284.5115966796875, 286.4185791015625, 286.4330749511719, 287.3827209472656, 287.7151794433594, 285.168701171875, 286.48773193359375, 287.0854187011719, 284.83477783203125, 285.6598205566406, 288.90838623046875, 289.5420837402344, 286.84149169921875, 287.568115234375, 288.66552734375, 286.90509033203125, 286.42449951171875, 287.4581298828125, 284.6081848144531, 286.8240661621094, 288.5355529785156, 289.1637878417969, 290.5512390136719, 289.5143127441406, 288.2817077636719, 286.84112548828125, 287.7165832519531, 285.4164123535156, 285.6025085449219, 288.2255859375, 284.05810546875, 285.99578857421875, 283.4344787597656, 284.9076232910156, 287.93316650390625, 289.7757263183594, 291.4762878417969, 287.8082580566406, 290.1181945800781, 287.81304931640625, 289.1233825683594, 285.1474914550781, 284.74359130859375, 284.2395324707031, 286.2467346191406, 287.7171325683594, 285.564697265625, 285.8836669921875)
VALUES: List(292.7082, 293.20105, 296.84433, 292.93924, 290.55487, 289.90823, 291.49646, 291.292, 290.19125, 290.29752, 291.30807, 287.21075, 289.83224, 284.07733, 291.28607, 290.5197, 293.05023, 290.01834, 290.62634, 296.0042, 294.7265, 296.67496, 293.814, 289.27795, 288.54807, 292.72394, 290.34406, 291.90012, 291.09796, 291.28616, 292.61237, 292.4619, 294.27078, 297.08112, 295.1556, 295.04993, 291.06168, 296.17084, 295.35748, 295.86917, 296.2025, 288.21942, 291.53064, 289.56323, 292.07068, 295.22385, 294.6864, 295.908, 296.1203, 291.01007, 287.0889, 287.0923, 284.78595, 282.5561, 283.19363, 288.502, 284.19522, 285.9576, 283.93478, 282.60132, 282.87802, 283.32504, 284.394, 287.34668, 286.561, 283.2228, 280.57468, 281.3531, 280.46478, 282.4374, 282.92157, 282.75208, 280.245, 280.18832, 277.37344, 277.9842, 277.45813, 282.215, 285.2746, 285.05444, 284.5563, 284.50632, 276.54214, 277.54868, 275.7071, 278.4978, 282.07388, 283.50284, 279.93674, 279.32132, 284.40698, 287.42154, 288.15417, 286.66046, 287.30093, 286.13257, 287.87347, 290.99512, 289.08908, 285.163, 284.5066, 286.70703, 286.60995, 287.74057, 287.49603, 285.70682, 283.85526, 286.8201, 291.79498, 291.9578, 292.48276, 292.44434, 288.41132, 286.54364, 285.9193, 285.1902, 285.11887, 287.35754, 287.8274, 288.621, 289.60965, 290.4715, 290.20667, 290.39404, 289.2234, 291.25583, 292.37253, 291.16898, 290.62042, 292.04428, 293.8359, 293.80576, 296.25058, 295.72098, 295.57996, 294.87143, 295.1048, 295.42032, 293.66525, 292.1078, 292.19812, 291.43097, 291.89062, 291.52768, 292.12955, 292.32904, 292.3408, 292.99487, 294.21936, 294.98447, 295.75952, 292.36444, 293.79315, 294.00284, 290.62445, 290.7234, 290.54483, 287.55356, 287.92523, 289.1805, 291.07358, 291.33682, 293.49844, 294.122, 295.4025, 295.10294, 294.0093, 290.12012, 289.3953, 290.38116, 289.78827, 290.81512, 292.3837, 293.0703, 292.65634, 289.352, 289.2106, 291.70987, 291.5855, 290.71106, 291.27597, 293.77444, 291.83145, 292.1063, 291.1015, 292.64685, 291.3985, 288.7875, 291.2516, 291.84497, 292.37762, 294.574, 292.80923, 292.5181, 294.57642, 296.4698, 297.66846, 296.47415, 295.13315, 291.31305, 295.3933, 294.02774, 294.91336, 293.86746, 294.06027, 291.96527, 291.30692, 291.1941, 293.06006, 293.3532, 294.10724, 291.76495, 295.04996, 294.57626, 294.9208, 295.85254, 297.24774, 297.10165, 295.87958, 295.84915, 293.86728, 294.14914, 295.72595, 295.79587, 295.3897, 295.93173, 298.09106, 297.82486, 298.10522, 298.95215, 298.3263, 299.52817, 299.36606, 299.19855, 297.32333, 296.95544, 296.01505, 295.78336, 297.02927, 297.32397, 296.87515, 296.29712, 295.26508, 294.9322, 295.31927, 296.59583, 298.04724, 298.70258, 298.3352, 297.6646, 297.3118, 296.7875, 296.68826, 297.73502, 296.27975, 293.2102, 290.19574, 292.1457, 289.3529, 288.66068, 291.7725, 293.3418, 291.8747, 291.3999, 293.42694, 297.47583, 296.3548, 293.00162, 291.7663, 289.30554, 289.2756, 288.937, 291.30762, 293.3711, 289.52914, 291.74707, 291.93985, 289.35147, 287.9378, 288.25085, 290.263, 292.05115, 293.5296, 291.9442, 294.25888, 292.1776, 290.79984, 291.13263, 291.7488, 290.97223, 288.5675, 289.09137, 288.83603, 285.5859, 284.76135, 284.35907, 284.11163, 284.22147, 287.67065, 287.23438, 282.6923, 279.8508, 284.07556, 284.67606, 285.38174, 286.37778, 286.08887, 286.47464, 286.70642, 285.91388, 283.98767, 281.90436, 285.77673, 286.27475, 285.19965, 284.1803, 280.55594, 280.82364, 280.98816, 281.21646, 278.43198, 279.00623, 280.40356, 282.50238, 282.23962, 283.39783, 278.1809, 280.26306, 283.2497, 279.54895, 279.90106, 287.36557, 286.81598, 285.8459, 287.45813, 282.3802, 284.8254, 285.99704, 286.0373, 288.16797, 286.56876, 283.9962, 286.10532, 285.80115, 283.52118, 285.4414, 281.40433, 281.72913, 280.09964, 277.39383, 279.9409, 283.61243, 285.27164, 291.4763, 285.9355, 286.3141, 287.81305, 286.07498, 284.3713, 282.11002, 280.8152, 281.6342, 283.81134, 283.137, 279.98343)
```

Here are the time series in graphical form.
(The code in this project did not produce graph, just the data.)
![plot](https://cloud.githubusercontent.com/assets/11281373/26793804/59526952-49ed-11e7-8edd-b3b68e5036b0.png)

This program should take about 10 seconds to complete.

## Example 2 (File on S3) ##

We can re-run the same example, this time reading the data directly from S3:

```bash
$SPARK_HOME/bin/spark-submit --master 'local[*]' \
   gddp/target/scala-2.11/gddp-assembly-0.22.7.jar \
   's3://nasanex/NEX-GDDP/BCSD/rcp85/day/atmos/tasmin/r1i1p1/v1.0/tasmin_day_BCSD_rcp85_r1i1p1_inmcm4_2099.nc' \
   ./geojson//CA.geo.json \
   '33.897,-118.225'
```

The output will be the same as above, but the process will take longer to complete -- generally about 2 minutes and 40 seconds in my experience.
(Of course, this will vary with connection speed and proximity to the S3 bucket.)

Measurements show that about 190 - 200 megabytes are downloaded in total.
Two separate sweeps are taken through the file (one to produce the clipped tiles and one to do point reads), so effectively the cost of taking one pass through the file reading <= 32 kilobyte subsets of each tile is about 95 - 100 megabytes.
(The file is 767 megabytes, the amount downloaded varies with the number of executors used.)

There is still room for improvement in the S3 functionality, that is [discussed below](#future-work-better-s3-caching).

# Structure Of This Code #

## Open ##

[This code](gddp/src/main/scala/Gddp.scala#L37-L47):
```scala
def open(uri: String) = {
  if (uri.startsWith("s3:")) {
    val raf = new ucar.unidata.io.s3.S3RandomAccessFile(uri, 1<<15, 1<<24)
    NetcdfFile.open(raf, uri, null, null)
  } else {
    NetcdfFile.open(uri)
  }
}
```
is where the local or remote NetCDF file is opened.

Notice that there are different code paths for S3 versus other locations; that is not strictly necessary and is done for efficiency.
`NetcdfFile.open("s3://...")` returns a perfectly working `NetcdfFile`, but its underlying buffer size and cache behavior are not optimal for GDDP.
That is further discussed [below](#future-work-better-s3-caching).

## Whole Tile Read ##

[This code](gddp/src/main/scala/Gddp.scala#L65-L78):
```scala
val ncfile = open(netcdfUri)
val vs = ncfile.getVariables()
val ucarType = vs.get(1).getDataType()
val latArray = vs.get(1).read().get1DJavaArray(ucarType).asInstanceOf[Array[Float]]
val lonArray = vs.get(2).read().get1DJavaArray(ucarType).asInstanceOf[Array[Float]]
val attribs = vs.get(3).getAttributes()
val nodata = attribs.get(0).getValues().getFloat(0)
val wholeTile = {
  val tileData = vs.get(3).slice(0, 0)
  val Array(y, x) = tileData.getShape()
  val array = tileData.read().get1DJavaArray(ucarType).asInstanceOf[Array[Float]]
  FloatUserDefinedNoDataArrayTile(array, x, y, FloatUserDefinedNoDataCellType(nodata)).rotate180.flipVertical
}
```
is where the whole first tile (tile for day zero) is read into a GeoTrellis tile.

Notice the assignments `latArray = vs.get(1)...`, `lonArray = vs.get(2)...`, and `tileData = vs.get(3)...`.
These are enabled by prior knowledge that in GDDP files, the variable with index 1 is an array of latitudes, the variable with index 2 is an array of longitudes, and the variable with index 3 is three-dimensional temperature data.
Perhaps a more robust way to do this would be to iterate through the list of variables and match against the string returned by `vs.get(i).getFullName()`, but the present approach is good enough for government work.

The dimensions of the temperature data are time (in units of days), latitude, and longitude, in that order.
`vs.get(3).slice(0, 0)` requests a slice with the first (0th) index of the first dimension fixed; it requests the whole tile from the first (0th) day.

The assignment `nodata = attribs.get(0).getValues().getFloat(0)` gets the "fill value" for the temperature data which is used as the `NODATA` value for the GeoTrellis tile that is constructed.

## Partial Tile Read ##

[This code](gddp/src/main/scala/Gddp.scala#L132-L135):
```scala
val array = tasmin
  .read(s"$t,$ySliceStart:$ySliceStop,$xSliceStart:$xSliceStop")
  .get1DJavaArray(ucarType).asInstanceOf[Array[Float]]
FloatUserDefinedNoDataArrayTile(array, x, y, FloatUserDefinedNoDataCellType(nodata)).rotate180.flipVertical
```
is where the partial tiles (matched to the extent of the query polygon) are read.
Here, instead of using the `slice` method, the slicing functionality of the `read` method is used.
The string that results from `s"$t,$ySliceStart:$ySliceStop,$xSliceStart:$xSliceStop"` will contain three descriptors separated by commas;
the first is a time (specified by an integral index), the second is a latitude range (a range of integral indices with the start and end separated by a colon) and the last is a longitude range (again, a range of integral indices).

## Point Read ##

[This code](gddp/src/main/scala/Gddp.scala#L158-L160):
```scala
tasmin
  .read(s"$t,$ySlice,$xSlice")
  .getFloat(0)
```
uses the slicing capability of the `read` method to get the temperature value at a particular time, at a particular latitude, and at a particular longitude.

# Future Work: Better S3 Caching #

As mentioned above [here](#example-2-file-on-s3) and [here](#open), the efficiency of reading from S3 is an area for future improvement.

There are many different types of files that have the extension `.nc`, in this brief discussion we will restrict attention the GDDP dataset;
those files store the temperature data in compressed chunks of roughly 32 KB in size, and those chunks are indexed by a [B-tree](https://en.wikipedia.org/wiki/B-tree) with internal nodes of size 16 KB.

Within the S3 reader, a simple FIFO cache has been implemented.
The `S3RandomAccessFile` implementation emulates and a random access file, and for compatibility must maintain a read/write buffer.
For simplicity, the cache block size used is twice the size of that buffer.

When one makes a call of the form `NetcdfFile.open("s3://...")`, the buffer size used is 512 KB which results in 1 MB cache blocks.
That implies that every request to read an uncached datum incurrs a 1 MB download.
For many access patterns that is not a problem -- and in fact is the intended behavior -- but reading small windows or single pixels from different daily tiles is not a good access pattern for this scheme.
Such a pattern would result in a lot of extra data being downloaded and cached when accessing leaves, as well as the eviction of cache blocks containing internals nodes (which presumably may be revisited) in favor of cache blocks containing single-use leaf nodes.
The special-case that contains the line `raf = new ucar.unidata.io.s3.S3RandomAccessFile(uri, 1<<15, 1<<24)` creates an `S3RandomAccessFile` with a 32 KB read/write buffer and 64 KB cache blocks.
The 64 KB cache blocks are a much better fit than the 1 MB ones that would have been created by default.

Examination of the access patterns generated by point queries, e.g. `read("107,22,7")`, shows that accesses are in general not aligned to block boundaries which frequently results in two blocks being fetched when one would have done.
There is also the question of unnecessary cache evication that was raised above.
One of many ideas for improvement is to implement a cache that pays attention to the particular structure of the file rather than just managing blocks of bytes as the current one does.
That can potentially be done by inspecting the program state before a read is performed to determine whether the request is for an internal node or a leaf.
If it is an internal node, the entire node can be cached (or returned from the cache), and if it is a leaf the request can be serviced without involving the cache.
