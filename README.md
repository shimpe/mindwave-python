                                                                            
      This file is part of mindwave-python.
  
      mindwave-python is free software: you can redistribute it and/or modify
      it under the terms of the GNU General Public License as published by
      the Free Software Foundation, either version 3 of the License, or
      (at your option) any later version.
  
      mindwave-python is distributed in the hope that it will be useful,
      but WITHOUT ANY WARRANTY; without even the implied warranty of
      MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
      GNU General Public License for more details.
  
      You should have received a copy of the GNU General Public License
      along with mindwave-python.  If not, see <http://www.gnu.org/licenses/>.
                                                                            
mindwave-python
===============

Linux and Mac-friendly Python parser to connect and interact with multiple NeuroSky MindWave headsets from one machine.

This program is based on the software used by Moonshot Lab at Barkley (http://moonshot.barkleyus.com/) for all of their internal MindWave projects.

Stefaan Himpe added parser support for asic eeg power and raw wave signals and improved checksum checking. He also added some visualization tools to explore the brain waves under several transforms.

Running the sample applications
-------------------------------

Get the code and put it in some folder, e.g. `/home/your_name/mindwave-python`
Then, from that folder run one of `scope.py`, `eyeblink.py` or `wavelet.py`.
The sample applications have additional dependencies on PyQt4, pyqtgraph, numpy and pywavelet.


Basic Parser Usage
------------------

A connection to the headset dongle is established by creating a new `Headset` object. Find the MindWave device(s) on your machine. On a Mac, it looks like this:

    import mindwave
    
    headset = mindwave.Headset('/dev/tty.MindWave', '625f')

Pass in the device path and the headset ID printed inside the battery case.

It's recommended to wait at least a couple seconds before connecting the dongle to the headset:

    import mindwave, time
    
    headset = mindwave.Headset('/dev/tty.MindWave', '625f')
    time.sleep(2)
    
    headset.connect()
    print "Connecting..."
    
    while headset.status != 'connected':
        time.sleep(0.5)
        if headset.status == 'standby':
            headset.connect()
            print "Retrying connect..."
    print "Connected."
    
    while True:
        print "Attention: %s, Meditation: %s" % (headset.attention, headset.meditation)

For the MindWave Mobile bluetooth headsets, it's unnecessary to use the `connect()` or `disconnect()` methods. If your operating system automatically creates a serial port for the bluetooth device, there's also no need to specify a headset ID. Just pass the serial device path as a parameter when you create a new `Headset` object and you're good to go.


Auto-connect
------------

The library can also auto-connect the dongle to the first available headset, rather than specifying a headset ID.

    import mindwave, time
    
    headset = mindwave.Headset('/dev/tty.MindWave')
    time.sleep(2)
    headset.connect()

Use `headset.autoconnect()` to auto-connect explicitly, regardless of whether or not a headset ID was specified.


Multiple headsets
-----------------

The library can handle multiple devices used simultaneously.

    import mindwave, time
    
    h1 = mindwave.Headset('/dev/tty.MindWave', '625f')
    h2 = mindwave.Headset('/dev/tty.MindWave2', 'a662')
    time.sleep(2)
    
    h1.connect()
    print "Connecting h1..."
    while h1.status != 'connected':
        time.sleep(0.5)
        if h1.status == 'standby':
            h1.connect()
            print "Retrying connect..."
    print "Connected h1."
    
    h2.connect()
    print "Connecting h2..."
    while h2.status != 'connected':
        time.sleep(0.5)
        if h2.status == 'standby':
            h2.connect()
            print "Retrying connect..."
    print "Connected h2."
    
    while True:
        print "Attention 1: %s, Meditation 1: %s" % (h1.attention, h1.meditation)
        print "Attention 2: %s, Meditation 2: %s" % (h2.attention, h2.meditation)


Adding event handlers
---------------------

The library provides hooks for certain events to allow for the attachment of custom handlers.

    def on_blink(headset, blink_strength):
        print "Blink detected. Strength: %s" % blink_strength
    
    headset.blink_handlers.append(on_blink)


API
===

Available properties
--------------------

`headset.` **device** - The device path of the dongle on the system.

`headset.` **headset_id** - The ID of the connected headset.

`headset.` **poor_signal** - The "poor signal" reading of the headset connection. This indicates how poorly the headset is reading EEG waves, 0 being the best reading and 255 being the worst. Try readjusting the headset if this value is too high.

`headset.` **attention** - The last-read attention value from the headset.

`headset.` **meditation** - The last-read meditation value from the headset.

`headset.` **blink** - The last-read blink strength from the headset.

`headset.` **status** - The current status of the headset: `connected`, `scanning`, or `standby`


Available methods
-----------------

`headset.` **connect** `([headset_id])` - Connect to the specified headset ID. If no headset was specified, the dongle will auto-connect to the first available.

`headset.` **autoconnect** `()` - Auto-connect to the first available headset, regardless of any headset ID specified.

`headset.` **disconnect** `()` - Disconnect the dongle from the headset.


Event hooks
-----------

`headset.` **poor_signal_handlers** `[]` - Handlers are fired whenever a poor signal is detected. Expects handlers with the prototype `my_function(headset, poor_signal)` and passes in the current headset object and poor signal value.

`headset.` **good_signal_handlers** `[]` - Handlers are fired whenever a good signal is detected. Expects handlers with the prototype `my_function(headset, poor_signal)` and passes in the current headset object and poor signal value.

`headset.` **attention_handlers** `[]` - Handlers are fired whenever an attention value is received. Expects handlers with the prototype `my_function(headset, attention)` and passes in the current headset object and attention value.

`headset.` **meditation_handlers** `[]` - Handlers are fired whenever an meditation value is received. Expects handlers with the prototype `my_function(headset, meditation)` and passes in the current headset object and meditation value.

`headset.` **blink_handlers** `[]` - Handlers are fired whenever a blink is detected. Expects handlers with the prototype `my_function(headset, blink_strength)` and passes in the current headset object and blink strength value.

`headset.` **connected_handlers** `[]` - Handlers are fired whenever the headset is connected. Expects handlers with the prototype `my_function(headset)` and passes in the current headset object.

`headset.` **eeg_power_handler** `[]` - Handlers are fired whenever a raw wave value is sent. Expects handlers with the prototype `my_function(headset, delta, theta, lowalpha, highalpha, lowbeta, highbeta, lowgamma, midgamma)` and passes in the current headset object and the delta, theta, lowalpha, highalpha, lowbeta, highbeta, lowgamma and midgamma values.

`headset.` **raw_wave_handlers** `[]` - Handlers are fired whenever a raw wave value is sent. Expects handlers with the prototype `my_function(headset, raw_wave_value)` and passes in the current headset object and the raw wave value.

`headset.` **chksum_mismatch_handlers** `[]` - Handlers are fired whenever a checksum check fails. Expects handlers with the prototype `my_function(headset, expected_checksum, actual_checksum)` and passes in the current headset object and expected and actual checksum.

`headset.` **notfound_handlers** `[]` - Handlers are fired whenever the headset specified cannot be found. Expects handlers with the prototype `my_function(headset, not_found_id)` and passes in the current headset object and the headset id that could not be found.

`headset.` **disconnected_handlers** `[]` - Handlers are fired whenever the headset is disconnected. Expects handlers with the prototype `my_function(headset)` and passes in the current headset object.

`headset.` **request_denied_handlers** `[]` - Handlers are fired whenever a request to the dongle is denied (connect/disconnect/autoconnect). Expects handlers with the prototype `my_function(headset)` and passes in the current headset object.

`headset.` **scanning_handlers** `[]` - Handlers are fired whenever the dongle begins scanning for a headset. Expects handlers with the prototype `my_function(headset)` and passes in the current headset object.

`headset.` **standby_handlers** `[]` - Handlers are fired whenever the dongle goes into standby (not connected to a headset). Expects handlers with the prototype `my_function(headset)` and passes in the current headset object.

Changelog:
==========

- 10 may 2013: remove duplicated module and update README
- 08 may 2013: more improvements to code quality
- 07 may 2013: small improvements to code quality
- 07 may 2013: add proof of concept mind-controlled mouse replacement
- 05 may 2013: add gplv3 LICENSE
- 05 may 2013: fft based eye blink detection
- 05 may 2013: important bug-fixes in the data parser (this breaks the naive eye blink detector)
- 04 may 2013: added two visualization tools
- 01 may 2013: added support for asic eeg power values, raw wave values, and proper checksum checking

