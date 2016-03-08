The following error message occurs when processing "raw" formatted TDMS files from a cDAQ. A problematic TDMS file is located here: `./exampleFiles/Raw000.tdms` . 

```
Converting 'rawEngineData.tdms'...Error using zeros
NaN and Inf not allowed.

Error in convertTDMS>getData (line 1156)
            ob.(cname).data=zeros(nsamples,1);

Error in convertTDMS (line 338)
    ob=getData(fid,channelinfo,SegInfo);  %Returns the objects which have data.  See postProcess function (appends to all of the objects)
```

Every other TDMS parser (excel, python) we've tried seems to not have an issue -- here we are using Excel as a reference. We've consulted with the NI white paper, but haven't been able to get this fixed yet. We do not directly generate the TDMS file, so we do not know the datatype logged.

We we feel the the way the datatype's are read out is likely incorrect because when you process the file with the excel TDMS parser or python TDMS parser, the datatype should be a `float` (type 9) and not a `int16` (type 2) on the logged channels like ("`Synchronous/Air Manifold Pressure`" found at `output.Data.MeasuredData(15).Data`). That channel should have a raw value at an average of 1.975 instead of 6191. The multiplier also goes to Inf because of a divide by near zero up in `getChanInfo()`. Not sure which error comes first?

Noting that the `datastartindex` is 0 in the middle of the vector, we attempted to remove it removing that and replacing it with something else helps the function continue parsing. That is the modification made in the `hotfix/issue` branch on github. With that modification, the length of the data is not the same as other parsers read but does capture the general shape of the data when plotted (although the wrong data types still).

Any ideas? Any suggested way of debugging this issue?

Thanks, 
Jacob