local LuckyDataStore = require(7239753643)

local default = {
	Money = 100,
	Experience = 0,
	Rank = "None",
    Tools = {}, -- Tables should not be used on the combine function as they are not a variable.
}

game.Players.PlayerAdded:Connect(function(player)
	
	local folder = Instance.new("Folder")
	folder.Name = "leaderstats"
	
	local money = Instance.new("IntValue",folder)
	money.Name = "Money"
	
	local experience = Instance.new("IntValue",folder)
	experience.Name = "Experience"
	
	local rank = Instance.new("StringValue",folder)
	rank.Name = "Rank"
	
	local playerData = LuckyDataStore.CreateData(player,"Test",default)
	playerData:Combine(money,experience,rank)

    folder.Parent = player
end)
