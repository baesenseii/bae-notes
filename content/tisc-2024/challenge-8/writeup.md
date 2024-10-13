---
title: "How to publish Obsidian notes with Quartz on GitHub Pages"
draft: true
tags:
  - 
---
![[Pasted image 20240930055803.png]]
# part 1

![[Pasted image 20240930055643.png]]
![[Pasted image 20240930055650.png]]

Used frida to hook the ByteBuffer wrap function, which takes in the byte[] of the decrypted data
```
Java.perform(function() {

// get the bytebuffer data - which is the final output of the K function.
var className = "java.nio.ByteBuffer";
var classInstance = Java.use(className);

classInstance.wrap.overload("[B").implementation = function(arg1)
{
    // create a Base64 object first
    var b64_obj = Java.use("android.util.Base64").$new();
    console.log(b64_obj.encodeToString(arg1,2));

    
    return this.wrap(arg1);
};
```

# Part 2

Final Frida Script Code. How to use it:
- save script into a file
- Run the following Frida syntax:
```
frida -U -f com.wall.facer -l script.js
```
- Type in the text "I am a tomb" in the application and click 'submit' (adb logcat logs will show that the native library is loaded).
- Open your script.js in a notepad and just click 'save'. Frida assumes that there are changes made if there's a file operation, which will refresh the script again.
- Once done, go to the app and type 'Only Advance'. Should be presented with key and iV.
```
const baseAddr = Module.findBaseAddress("libnative.so");
console.log("Base Address: "+baseAddr);

// knocking down wall 1
var pc0 = new NativePointer(baseAddr.add(0x5ab0));
console.log(Memory.protect(pc0,15,'rw-'));
Memory.writeByteArray(pc0,[ 0x2f, 0x64, 0x61, 0x74, 0x61, 0x2f, 0x6c, 0x6f, 0x63, 0x61, 0x6c, 0x2f, 0x63, 0x61, 0x62 ]); //write to replacement string /data/local/cab (just created a sample file in the phone)
// this is to also preserve the same number of bytes
console.log("[WALL1] Replacing '/sys/wall/facer' to '/data/local/cab'..");

// for patching instruction to knock down wall 2
var pc1 = new NativePointer(baseAddr.add(0x1f78));
Memory.protect(pc1,5,'rwx');
Memory.writeByteArray(pc1,[0xbf, 0x39, 0x5, 0x0, 0x0]);
console.log("Patching instruction at offset 0x1f78...");

//Knocking down of Wall 3
//2a looks good => removed the "i'm afraid i'm going to have to stop you from getting the correct key and IV"
var pc2a = new NativePointer(baseAddr.add(0x231a));
console.log("Patching instruction at offset 0x231a...");
Memory.protect(pc2a,6,'rwx');
Memory.writeByteArray(pc2a,[0xc7, 0xc7, 0x39, 0x5,0x0,0x0]);

//2b looks good => this is to patch the other switch case that uses 0xa13 instead of the usual 0x539 for others

var pc2b = new NativePointer(baseAddr.add(0x3746));
console.log("Patching instruction at offset 0x3746...");
Memory.protect(pc2b,6,'rwx');
Memory.writeByteArray(pc2b,[0xc7, 0xc1, 0x39, 0x5,0x0,0x0]);
```