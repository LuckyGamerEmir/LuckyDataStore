--[[
    LuckyDataStore by LuckyGamerEmir
--]]

-- Signals
local SuccessfullyLoadedSignal = Instance.new("BindableEvent")
local ErrorOnLoadingDataSignal = Instance.new("BindableEvent")

local LuckyDataStore = {
	-- Settings
	AutoSaveEnabled = true,
	AutoSaveCooldown = 180,
	DebugMessages = true,
	SaveInStudio = true,
	EnableBindToClose = true,
	-- Events
	SuccessfullyLoaded = SuccessfullyLoadedSignal.Event,
	ErrorOnLoadingData = ErrorOnLoadingDataSignal.Event,
}

local DataStoreService = game:GetService("DataStoreService")
local RunService = game:GetService("RunService")

local DataBase = {}
local PlayerData = {}
PlayerData.__index = PlayerData

local function Copy(original)
	local copy = {}
	for key,value in pairs(original) do
		copy[key] = value
	end
	return copy
end

local function WaitForRequestBudget()
	local currentBudget = DataStoreService:GetRequestBudgetForRequestType(Enum.DataStoreRequestType.UpdateAsync)
	while currentBudget < 1 do
		currentBudget = DataStoreService:GetRequestBudgetForRequestType(Enum.DataStoreRequestType.UpdateAsync)
		task.wait(5)
	end
end

local function SendMessage(message)
	if LuckyDataStore.DebugMessages then
		warn(message)
	end
end

LuckyDataStore.CreateData = function(player,key,default)
	assert(typeof(player) == "Instance" and player.ClassName == "Player","Argument 1 must be a player.")
	assert(typeof(key) == "string","Argument 2 must be a string.")
	assert(typeof(default) == "table","Argument 3 must be a table.")
	default["SessionLock"] = false
	local self = setmetatable({Data = nil, Player = player, Key = key, Default = Copy(default)},PlayerData)
	self:GetData()
	table.insert(DataBase,self)
	return self
end

LuckyDataStore.FindData = function(player,key)
	assert(typeof(player) == "Instance" and player.ClassName == "Player","Argument 1 must be a player.")
	assert(typeof(key) == "string","Argument 2 must be a string.")
	for _,data in pairs(DataBase) do
		if data.Player.UserId == player.UserId and data.Key == key then
			return data
		end
	end
	warn("Couldn't find data for player "..player.Name..".")
	return nil
end

function PlayerData:Combine(...)
	if #{...} == 0 then return end
	if not self.Data then return end
	for _,variable in pairs({...}) do
		variable.Value = self.Data[variable.Name]
		variable.Changed:Connect(function(newvalue)
			self.Data[variable.Name] = newvalue
		end)
	end
end

function PlayerData:GetData()
	local success, err
	repeat task.wait()
		WaitForRequestBudget()
		success, err = pcall(function()
			DataStoreService:GetDataStore(self.Key):UpdateAsync("Player_"..self.Player.UserId,function(oldData)
				self.Data = oldData or Copy(self.Default)
				for index,value in pairs(self.Default) do
					if self.Data[index] == nil then
						self.Data[index] = value
					end
				end
				if self.Data.SessionLock then
					if os.time() - self.Data.SessionLock < 1800 then
						err = "Wait"
					else
						self.Data.SessionLock = os.time()
						return self.Data
					end
				else
					self.Data.SessionLock = os.time()
					return self.Data
				end
			end)
			if err == "Wait" then
				task.wait(10)
			end
		end)
	until success or not self.Player.Parent
	if not self.Player.Parent then
		ErrorOnLoadingDataSignal:Fire(self.Player.Name,"Player left while trying to load their data.")
		return
	end
	if typeof(self.Data) == "table" then
		SendMessage("Successfully loaded data for player "..self.Player.Name..".")
		SuccessfullyLoadedSignal:Fire(self.Player.Name,self.Data)
	else
		warn(self.Player.Name.."'s data had an error while loading and will not be saved.")
		self.Data = Copy(self.Default)
		self.Data["Error"] = err or "Unknown"
		ErrorOnLoadingDataSignal:Fire(self.Player.Name,err or "Unknown")
	end
end

function PlayerData:Save(dontLeave,dontWait)
	if not self.Data then return end
	if not LuckyDataStore.SaveInStudio and RunService:IsStudio() then 
		SendMessage("Data has not been saved because SaveInStudio is false.")
		return
	end
	if self.Data["Error"] ~= nil then
		warn(self.Player.Name.." data has not been saved due to an error while loading data. Error: "..self.Data.Error)
		return
	end
	local success
	repeat
		if not dontWait then
			WaitForRequestBudget()
		end
		success = pcall(function()
			DataStoreService:GetDataStore(self.Key):UpdateAsync("Player_"..self.Player.UserId,function()
				self.Data["SessionLock"] = dontLeave and os.time() or nil
				return self.Data
			end)
		end)
	until success
	SendMessage("Successfully saved "..self.Player.Name.."'s data.")
end

local function PlayerRemoving(player)
	if RunService:IsStudio()then return end
	for _,data in pairs(DataBase) do
		if data.Player.UserId == player.UserId then
			data:Save()
			table.remove(DataBase,table.find(DataBase,data))
		end
	end
end

local function BindToClose()
	if not LuckyDataStore.SaveInStudio and RunService:IsStudio() then
		SendMessage("Data has not been saved because SaveInStudio is false.")
		return
	end
	if LuckyDataStore.EnableBindToClose then
		for _,data in pairs(DataBase) do
			data:Save(nil,true)
		end
	end
end

if LuckyDataStore.AutoSaveEnabled then
	coroutine.wrap(function()
		while task.wait(LuckyDataStore.AutoSaveCooldown) do
			SendMessage("Starting an auto-save...")
			if not LuckyDataStore.SaveInStudio and RunService:IsStudio() then
				SendMessage("Stopped auto-save because SaveInStudio is false.")
				return 
			end
			for _,data in pairs(DataBase) do
				data:Save(true)
				SendMessage(data.Player.Name.."'s data has been auto-saved.")
			end
			SendMessage("Finished auto-save.")
		end
	end)()
end

game.Players.PlayerRemoving:Connect(PlayerRemoving)
game:BindToClose(BindToClose)

return LuckyDataStore
