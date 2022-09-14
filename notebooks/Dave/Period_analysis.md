# Basic period analysis #


```python
import numpy as np
import matplotlib.pyplot as plt
from astropy.timeseries import LombScargle
%matplotlib inline
```

First we need to download the data that we will search for periodicities:
[lc.data](https://iacerasmus.github.io/ERASMUS2022/data/lc.data)

Read-in the data and plot


```python
times,flux,fluxerr = np.loadtxt('lc.data',unpack=True)
plt.errorbar(times,flux,yerr=fluxerr,fmt='k.')
plt.xlabel('time')
plt.ylabel('flux')
```




    Text(0, 0.5, 'flux')




    
![png](Period_analysis_files/Period_analysis_4_1.png)
    


Now let's run a Lomb-Scargle period analysis based on the implementation of Press & Rybicki (1989)


```python
ls = LombScargle(times, flux, fluxerr)
frequency, power = ls.autopower()
one_percent=ls.false_alarm_level(0.01)
plt.plot(frequency,power)
plt.axhline(one_percent,ls='--')
plt.ylabel("LS power")
plt.xlabel("Freq")
```




    Text(0.5, 0, 'Freq')




    
![png](Period_analysis_files/Period_analysis_6_1.png)
    


Let's plot again, but zoom in on the region of interest


```python
plt.plot(frequency,power)
plt.axhline(one_percent,ls='--')
plt.ylabel("LS power")
plt.xlabel("Freq")
plt.xlim([0,5])
```




    (0.0, 5.0)




    
![png](Period_analysis_files/Period_analysis_8_1.png)
    


It seems pretty clear that the lower peaks are aliases of the strongest peak


```python
peak=frequency[np.where(power==power.max())]
plt.plot(frequency,power)
for x in range(0,5):
    plt.axvline(x*peak,ls='--')
plt.ylabel("LS power")
plt.xlabel("Freq")
plt.xlim([0,5])
```




    (0.0, 5.0)




    
![png](Period_analysis_files/Period_analysis_10_1.png)
    


Let's see what the light curve looks like folded on this period


```python
period=1/peak
phases=(times%period)/period
plt.errorbar(phases,flux,yerr=fluxerr,fmt='k.')
plt.xlabel("Phase")
plt.ylabel("Flux")
```




    Text(0, 0.5, 'Flux')




    
![png](Period_analysis_files/Period_analysis_12_1.png)
    


Quite clearly, we have two eclipses that have been phased up together.  So, the period is actually double this value.
If the eccentricity were greater, then this would be more obvious.


```python
period=2/peak
phases=(times%period)/period
plt.errorbar(phases,flux,yerr=fluxerr,fmt='k.')
plt.xlabel("Phase")
plt.ylabel("Flux")
```




    Text(0, 0.5, 'Flux')




    
![png](Period_analysis_files/Period_analysis_14_1.png)
    


Here we've stumbled across only one of the issues with assuming the strongest peak is reflective of the true periodicity.  There are many others which will depend on the object and your data, so never blindly accept the result!

Perhaps the most important issue conceptually, is the idea of the window function - or what periodicity would be identified just based on the sampling of your data.  You can quickly and easily inspect this by replacing your data with a non-variable array


```python
windowls = LombScargle(times, np.ones(len(times)))
window_frequency, window_power = windowls.autopower()
plt.plot(frequency,power,c='b')
plt.plot(window_frequency,window_power,c='r')
plt.ylabel("LS power")
plt.xlabel("Freq")
plt.xlim([0.5,5])
plt.ylim([0,0.2])
```




    (0.0, 0.2)




    
![png](Period_analysis_files/Period_analysis_17_1.png)
    


Here it is clear that our period does not line up with a peak of the window function, so we're pretty safe. If this were not the case, we should probably explore other possible periodicities (or try to obtain more data to break possible degeneracies).

It is not the case with this synthetic data, but the window function for real data often shows very strong peaks around 1d (and its aliases). So, one should always be particularly wary of periods very close to 1d, 2d, 0.5d, etc.


