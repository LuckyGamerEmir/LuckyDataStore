# LuckyDataStore

LuckyDataStore is a powerful module that saves your data, has a session locking and an auto-saving system(You can disable it if you want!).

- ## Module
```
local LuckyDataStore = require(7239753643) -- Getting the module.
```
- ## Configuration
```
LuckyDataStore.EnableBindToClose = true
```
Enables or disables game:BindToClose() function, this function is here to handle unexpected server shutdowns. Disable it if you are going to make a BindToClose function yourself. Default is true. Disabled automatically if you are on studio to prevent data store queue filling.
```
LuckyDataStore.SaveInStudio = true
```
Enables or disables studio saving. Default is true.

```
LuckyDataStore.DebugMessages = true
```

You can enable or disable debug messages. Example: "Saved player data successfully.", Default is true.
```
LuckyDataStore.AutoSaveEnabled = true
```
 Auto-save is enabled or not. Default is true.
```
LuckyDataStore.AutoSaveCooldown = 180 
```
This is how long script will wait in seconds until an auto-save will made. Default is 180 and I do not recommend setting this under 120 seconds as it can go over the data store limits.

- # Signals

## LuckyDataStore.SuccessfullyLoaded
```
LuckyDataStore.SuccessfullyLoaded:Connect(function(player_name, player_data)
end)
-- Fires when a successful load happens and returns player name and player data.
```
## LuckyDataStore.ErrorOnLoadingDataSignal
```
LuckyDataStore.ErrorOnLoadingDataSignal:Connect(function(player_name, err)
end)
-- Fires when an error occurs and returns player name and error.
```

- # Functions

## LuckyDataStore.CreateData()

```
local default = {
	Money = 100,
	Experience = 0,
	Rank = "None",
    Tools = {},
}

local playerData = LuckyDataStore.CreateData(player_instance,key,default)

-- This function will create a table and return it with given player and key
-- and must be used only once with the same key, otherwise there will be 2 tables
-- with given key and that can cause data loss and problems on saving.

-- To retrieve the data you created with this function, use FindData().

--[[
    Data = {}, -- If there was a save returns that, if not returns default table.
    Player = player_instance,
    Key = key,
    Default = {} -- Default values table.
--]]
```
## LuckyDataStore.FindData()
```
local playerData = LuckyDataStore.FindData(player_instance,key)
-- If table exists function will return that table.
-- If table doesn't exists function will warn and return nil.
```
## *variables* playerData:Combine(...)
```
playerData:Combine(money,experience,rank)
--[[
    Combines all values into the created player data and 
    updates the table when this values changes.
--]]
```
## *void* playerData.Data
```
local data = playerData.Data

-- To access the tools table,
local tools_table = data.Tools

--[[
    Returns a table containing saved values.
--]]
```
## *void* playerData:Save()
```
--[[
    Forces a save. Could be used when player buys a currency.
--]]
```
- # Some Important Notes

- Module will automatically fill data if it doesn’t exists. For example if you only had “Money” on your default values and you add more onto it, for example Experience, module will automatically add Experience to the data and put it is default value.

- Editing players variables will automatically update table. However editing table will not change variables value.

***This module is still work in progress, bugs may occur. If you encounter any bugs or errors please let me know. This is not the final result and everything is up to change.***
