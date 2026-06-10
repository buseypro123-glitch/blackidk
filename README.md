local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Brainrots Auto-Farm V14",
   Icon = 0,
   LoadingTitle = "Brainrots Auto-Farm",
   LoadingSubtitle = "V14 - Rayfield Edition",
   ShowText = "Farm",
   Theme = "Default",
   ToggleUIKeybind = "K",
   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,
   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil,
      FileName = "BrainrotFarmV14"
   },
   Discord = {
      Enabled = false,
      Invite = "noinvitelink",
      RememberJoins = true
   },
   KeySystem = false,
   KeySettings = {
      Title = "Brainrots Farm",
      Subtitle = "Loading",
      Note = "No method of obtaining the key is provided",
      FileName = "Key",
      SaveKey = true,
      GrabKeyFromSite = false,
      Key = {"Hello"}
   }
})

-- ===================== CORE VARIABLES =====================
local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")
local tweenService = game:GetService("TweenService")
local virtualInput = game:GetService("VirtualInputManager")
local localPlayer = players.LocalPlayer

local slotsFolder = workspace:WaitForChild("Bases"):WaitForChild("Plot1"):WaitForChild("Slots")
local returnCoords = CFrame.new(257, 3374, -45)

local allowedPositions = {
	Vector3.new(-2177, 2786, 84),
	Vector3.new(-2229, 2786, 84),
	Vector3.new(-2229, 2786, -70),
	Vector3.new(-2176, 2786, -70)
}
local maxDistanceCheck = 80
local isAutoFarming = false
local powerEnabled = false

local brainrotValueTable = {
	["Strawberry Elephant"] = 1000, ["Meowl"] = 950, ["Headless Horseman"] = 900, ["Griffin"] = 850,
	["Hydra Dragon Cannelloni"] = 700, ["Dragon Gingerini"] = 650, ["Dragon Cannelloni"] = 600, ["Esok Sekolah"] = 550,
	["Tictac Sahur"] = 400, ["Ketupat Kepat"] = 350, ["Tang Tang Kelentang"] = 300, ["Garama"] = 250, ["Madundung"] = 250,
	["Graipuss Medussi"] = 100, ["Pipi Kiwi"] = 50, ["Trippi Troppi"] = 10
}

-- ===================== FARM LOGIC =====================
local function forceTrigger(prompt)
	if fireproximityprompt then
		fireproximityprompt(prompt)
	else
		prompt:InputBegan(Enum.UserInputType.MouseButton1)
		task.wait(0.02)
		prompt:InputEnded(Enum.UserInputType.MouseButton1)
	end
end

local function checkLocation(prompt)
	local part = prompt:FindFirstAncestorWhichIsA("BasePart") or prompt.Parent
	if part and part:IsA("BasePart") then
		for _, pos in ipairs(allowedPositions) do
			if (part.Position - pos).Magnitude <= maxDistanceCheck then
				return part
			end
		end
	end
	return nil
end

local function executeFarmCycle()
	for _, obj in ipairs(workspace:GetDescendants()) do
		if not isAutoFarming then break end
		if obj:IsA("ProximityPrompt") and obj.Enabled then
			local validPart = checkLocation(obj)
			if validPart then
				pcall(function()
					local character = localPlayer.Character
					local rootPart = character and character:FindFirstChild("HumanoidRootPart")
					if rootPart then
						rootPart.CFrame = validPart.CFrame
						task.wait(0.12)
						forceTrigger(obj)
						task.wait(0.08)
						rootPart.CFrame = returnCoords
						task.wait(0.12)
					end
				end)
			end
		end
	end
end

task.spawn(function()
	while true do
		task.wait(0.3)
		if isAutoFarming then
			executeFarmCycle()
		end
	end
end)

local function simulatePhysicalKeyE(prompt)
	pcall(function()
		local targetKey = prompt.KeyboardKeyCode or Enum.KeyCode.E
		virtualInput:SendKeyEvent(true, targetKey, false, game)
		task.wait(prompt.HoldDuration + 0.1)
		virtualInput:SendKeyEvent(false, targetKey, false, game)
	end)
end

local function getToolPriorityValue(tool)
	local toolName = tool.Name
	if brainrotValueTable[toolName] then return brainrotValueTable[toolName] end
	for keyword, priority in pairs(brainrotValueTable) do
		if string.find(string.lower(toolName), string.lower(keyword)) then return priority end
	end
	return 0
end

local function equipBestInAllSlots()
	local character = localPlayer.Character
	if not character then return end
	local rootPart = character:FindFirstChild("HumanoidRootPart")
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if not (rootPart and humanoid) then return end

	local wasFarming = isAutoFarming
	isAutoFarming = false

	local slotsList = slotsFolder:GetChildren()
	table.sort(slotsList, function(a, b)
		local numA = tonumber(string.match(a.Name, "%d+")) or 0
		local numB = tonumber(string.match(b.Name, "%d+")) or 0
		return numA < numB
	end)

	for _, slot in ipairs(slotsList) do
		local targetPart = slot:IsA("BasePart") and slot or slot:FindFirstChildWhichIsA("BasePart", true)
		if targetPart then
			local bestBrainrot, highestPriority = nil, -1
			for _, tool in ipairs(localPlayer.Backpack:GetChildren()) do
				if tool:IsA("Tool") then
					local currentPriority = getToolPriorityValue(tool)
					if currentPriority > highestPriority then
						highestPriority = currentPriority
						bestBrainrot = tool
					end
				end
			end

			if bestBrainrot then
				humanoid:EquipTool(bestBrainrot)
				task.wait(0.08)
				rootPart.CFrame = targetPart.CFrame * CFrame.new(0, 3, 2)
				task.wait(0.2)
				local slotPrompt = slot:FindFirstChildOfClass("ProximityPrompt") or slot:FindFirstChildWhichIsA("ProximityPrompt", true)
				if slotPrompt and slotPrompt.Enabled then
					simulatePhysicalKeyE(slotPrompt)
					task.wait(0.15)
				else
					task.wait(0.1)
				end
			else
				Rayfield:Notify({Title = "Slots", Content = "Nenhum item elegível restante.", Duration = 3, Image = 4483362458})
				break
			end
		end
	end

	rootPart.CFrame = returnCoords
	Rayfield:Notify({Title = "Slots", Content = "Slots preenchidos com sucesso!", Duration = 3, Image = 4483362458})

	if wasFarming then
		isAutoFarming = true
	end
end

-- ===================== TELEPORT LOGIC =====================
local function teleportTo(x, y, z)
	local character = localPlayer.Character
	local rootPart = character and character:FindFirstChild("HumanoidRootPart")
	if rootPart then
		rootPart.CFrame = CFrame.new(x, y, z)
	end
end

local function teleportLastArea()
	local character = localPlayer.Character
	local rootPart = character and character:FindFirstChild("HumanoidRootPart")
	if not rootPart then return end

	rootPart.CFrame = CFrame.new(-26, 3374, 9)
	task.wait(0.2)

	local tweenInfo = TweenInfo.new(3, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
	local tween = tweenService:Create(rootPart, tweenInfo, {CFrame = CFrame.new(-2127, 2814, 6)})
	tween:Play()
	Rayfield:Notify({Title = "Teleport", Content = "Viajando para Last Area...", Duration = 3, Image = 4483362458})
end

-- ===================== RAYFIELD TABS =====================
local MainTab = Window:CreateTab("Farm", 4483362458)
local PowerTab = Window:CreateTab("Power", 4483362458)
local TeleportTab = Window:CreateTab("Teleport", 4483362458)

-- ── Main Tab ──
MainTab:CreateToggle({
	Name = "Auto Farm",
	CurrentValue = false,
	Flag = "AutoFarmToggle",
	Callback = function(Value)
		isAutoFarming = Value
		Rayfield:Notify({
			Title = "Auto Farm",
			Content = Value and "Farm ativado!" or "Farm desativado!",
			Duration = 2,
			Image = 4483362458
		})
	end
})

MainTab:CreateButton({
	Name = "Vender Agora",
	Callback = function()
		pcall(function()
			local customRemote = replicatedStorage:WaitForChild("Remotes", 1):WaitForChild("SellAllBrainrots", 1)
			if customRemote then customRemote:FireServer() end
		end)
		pcall(function()
			for _, descendant in ipairs(replicatedStorage:GetDescendants()) do
				if descendant:IsA("RemoteEvent") and string.find(string.lower(descendant.Name), "sell") then
					descendant:FireServer()
				end
			end
		end)
		Rayfield:Notify({Title = "Venda", Content = "Venda executada!", Duration = 2, Image = 4483362458})
	end
})

MainTab:CreateButton({
	Name = "Equipar Slots",
	Callback = function()
		task.spawn(equipBestInAllSlots)
	end
})

MainTab:CreateButton({
   Name = "Upgrade all brainrots once",
   Callback = function()
      task.spawn(function()
         for i = 1, 30 do
            -- Definimos as duas variáveis soltas em vez de uma tabela rígida
            local arg1 = 1
            local arg2 = i
            
            -- 1. Pede a informação em segundo plano para o loop NÃO travar se o servidor falhar
            task.spawn(function()
               pcall(function()
                  game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("GetUpgradeInfo"):InvokeServer(arg1, arg2)
               end)
            end)
            
            task.wait(0.05) -- Pequena pausa para o servidor registar o interesse
            
            -- 2. Envia o comando de compra com os argumentos diretos (sem unpack)
            pcall(function()
               game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("UpgradeBrainrot"):FireServer(arg1, arg2)
            end)
            
            task.wait(0.05) -- Pausa de 0.3 segundos (ideal para evitar sistemas de proteção anti-cheat)
         end
      end)
   end,
})



-- ── Power Tab ──
PowerTab:CreateSection("Stomp Power")

PowerTab:CreateToggle({
	Name = "+10 Power",
	CurrentValue = false,
	Flag = "Power10Toggle",
	Callback = function(Value)
		powerEnabled = Value
		if Value then
			task.spawn(function()
				while powerEnabled do
					replicatedStorage:WaitForChild("Remotes"):WaitForChild("BuyStomp"):FireServer(10)
					task.wait(0.05)
				end
			end)
		end
		Rayfield:Notify({
			Title = "+10 Power",
			Content = Value and "Power ativado!" or "Power desativado!",
			Duration = 2,
			Image = 4483362458
		})
	end
})


PowerTab:CreateToggle({
	Name = "+10 Power FAST (may cause lag)",
	CurrentValue = false,
	Flag = "Power10Togglefast",
	Callback = function(Value)
		powerEnabled = Value
		if Value then
			task.spawn(function()
				while powerEnabled do
					replicatedStorage:WaitForChild("Remotes"):WaitForChild("BuyStomp"):FireServer(10)
					task.wait(0.0000005)
				end
			end)
		end
		Rayfield:Notify({
			Title = "+10 Power",
			Content = Value and "Power ativado!" or "Power desativado!",
			Duration = 2,
			Image = 4483362458
		})
	end
})

-- ── Teleport Tab ──
TeleportTab:CreateSection("Áreas")

TeleportTab:CreateButton({
	Name = "Prison",
	Callback = function()
		teleportTo(454, 3379, 7)
		Rayfield:Notify({Title = "Teleport", Content = "Teleportado para Prison!", Duration = 2, Image = 4483362458})
	end
})

TeleportTab:CreateButton({
	Name = "Dungeon",
	Callback = function()
		teleportTo(331, 3383, -737)
		Rayfield:Notify({Title = "Teleport", Content = "Teleportado para Dungeon!", Duration = 2, Image = 4483362458})
	end
})

TeleportTab:CreateButton({
	Name = "Worlds",
	Callback = function()
		teleportTo(53, 3374, 81)
		Rayfield:Notify({Title = "Teleport", Content = "Teleportado para Worlds!", Duration = 2, Image = 4483362458})
	end
})

TeleportTab:CreateButton({
	Name = "Spider",
	Callback = function()
		teleportTo(339, 3345, 910)
		Rayfield:Notify({Title = "Teleport", Content = "Teleportado para Spider!", Duration = 2, Image = 4483362458})
	end
})

TeleportTab:CreateButton({
	Name = "Last Area",
	Callback = function()
		task.spawn(teleportLastArea)
	end
})

TeleportTab:CreateButton({
	Name = "moon",
	Callback = function()
		teleportTo(448, 2875, -11)
		Rayfield:Notify({Title = "Teleport", Content = "teleported to moon!", Duration = 2, Image = 4483362458})
	end
})
print("[Auto-Farm V14] Rayfield Edition carregado com sucesso.")
