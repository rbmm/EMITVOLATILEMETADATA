# EMITVOLATILEMETADATA
 

assume we build resource only dll (no code in it). let look on it map file

in **0** subdirectory:

let view on it MAP file , here exist `.rdata$voltmd` ( containing `__volatile_metadata` symbol) and `.rdata$zzzdbg` ( some debug information )

if look DLL in PEviewer it look like ok, resources ( `.rsrc` ) is visible. exist also `.rdata` section. 2 sections

assume we want remove debug info complete from PE ( `rdata$zzzdbg` ) (this can be apply to any PE)

for this exist undocumented option `/EMITPOGOPHASEINFO`. let build with it

in **1** subdirectory ( `/EMITPOGOPHASEINFO` ) ( wrong build !)

`.rdata$zzzdbg` is really gone. map file look like ok, bat if open DLL in PE viewer - we not view resources ! no `.rsrc` section at all !
if view DLL in binary - we can view that resources ( version in my case) still in DLL. but why it not visible in PE ?
answer in - `NumberOfSections`. it 1 ! now, but must be 2 - we still have `.rdata` ( `.rdata$voltmd`) and `.rsrc`
linker set wrong value for `NumberOfSections`

if we want remove `.rdata$voltmd` - for this exist another undocumented option - `/EMITVOLATILEMETADATA:NO`

in **2** ( `/EMITPOGOPHASEINFO /EMITVOLATILEMETADATA:NO`)

now we have only `.rsrc` section. .rcdata is gone. ( both `rdata$voltmd` and `.rdata$zzzdbg`)
and DLL again ok - `.rsrc` section is visible.
interesting that `__volatile_metadata` symbol still in map, but now it `<absolute>`, when in first case was `<linker-defined>`

so why was error in case **1** ?
linker add `__volatile_metadata symbol` to `.rdata` section, and if it still not exist - create it. but in forget increment `NumberOfSections` in this case.
if we already have `.rdata` section - all is ok ( `NumberOfSections` takes into account `.rdata`)
but if `__volatile_metadata` is single data in .rdata - we have wrong `NumberOfSections` value.

so if we use `/EMITPOGOPHASEINFO` option - we need or have some constant data (`.rdata`) in PE or use `/EMITVOLATILEMETADATA:NO` too

