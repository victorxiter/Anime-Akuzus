-- Torill Hub - UI sem bibliotecas externas (FrostWare compatível) local Players = game:GetService("Players") local TweenService = game:GetService("TweenService") local RunService = game:GetService("RunService") local LocalPlayer = Players.LocalPlayer local Mouse = LocalPlayer:GetMouse()

-- Variáveis Globais local selectedWorld = nil local selectedEnemy = nil local autoFarmEnabled = false local autoClickEnabled = false local enemiesList = {} local worldsList = {}

-- Funções auxiliares local function getWorlds() local enemiesFolder = workspace:FindFirstChild("Enemies") local list = {} if enemiesFolder then for _, world in ipairs(enemiesFolder:GetChildren()) do if world:IsA("Folder") then table.insert(list, world.Name) end end end return list end

local function getEnemies(worldName) local enemiesFolder = workspace:FindFirstChild("Enemies") local unique = {} if enemiesFolder and enemiesFolder:FindFirstChild(worldName) then for _, enemy in ipairs(enemiesFolder[worldName]:GetChildren()) do if enemy:IsA("Model") and not unique[enemy.Name] then unique[enemy.Name] = true end end end return unique end

local function getClosestEnemy(name) local folder = workspace.Enemies:FindFirstChild(selectedWorld) if not folder then return nil end

local closest, minDist = nil, math.huge
for _, enemy in ipairs(folder:GetChildren()) do
    if enemy:IsA("Model") and enemy.Name == name and enemy:FindFirstChild("HumanoidRootPart") then
        local dist = (enemy.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
        if dist < minDist then
            closest, minDist = enemy, dist
        end
    end
end
return closest

end

local function teleportTo(pos) if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(pos + Vector3.new(0, 5, 0)) end end

local function autoFarm() while autoFarmEnabled and selectedEnemy do local target = getClosestEnemy(selectedEnemy) if target and target:FindFirstChild("Humanoid") and target:FindFirstChild("HumanoidRootPart") then teleportTo(target.HumanoidRootPart.Position) repeat task.wait(0.1) until target.Humanoid.Health <= 0 or not autoFarmEnabled else task.wait(0.5) end end end

local function autoClick() while autoClickEnabled do mouse1click() task.wait(0.01) end end

-- Criar UI local ScreenGui = Instance.new("ScreenGui", game.CoreGui) ScreenGui.Name = "TorillHub" ScreenGui.ResetOnSpawn = false ScreenGui.IgnoreGuiInset = true

local MainFrame = Instance.new("Frame", ScreenGui) MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30) MainFrame.BackgroundTransparency = 0.25 MainFrame.Position = UDim2.new(0.05, 0, 0.1, 0) MainFrame.Size = UDim2.new(0, 500, 0, 300) MainFrame.Active = true MainFrame.Draggable = true MainFrame.BorderSizePixel = 0 MainFrame.ClipsDescendants = true MainFrame.Name = "MainFrame"

local UICorner = Instance.new("UICorner", MainFrame) UICorner.CornerRadius = UDim.new(0, 12)

-- Menu Lateral com ícones local SideBar = Instance.new("Frame", MainFrame) SideBar.Size = UDim2.new(0, 50, 1, 0) SideBar.Position = UDim2.new(0, 0, 0, 0) SideBar.BackgroundColor3 = Color3.fromRGB(20, 20, 20) SideBar.BorderSizePixel = 0

local function createTabButton(text, index) local button = Instance.new("TextButton", SideBar) button.Size = UDim2.new(1, 0, 0, 50) button.Position = UDim2.new(0, 0, 0, 50 * (index - 1)) button.Text = text button.TextColor3 = Color3.new(1, 1, 1) button.BackgroundTransparency = 1 button.Font = Enum.Font.SourceSansBold button.TextSize = 24 return button end

local tabs = { ["🏠"] = nil, ["🗡️"] = nil, ["🌟"] = nil, ["📍"] = nil, ["👉"] = nil }

local contentFrames = {} local currentTab = nil

local function switchTab(tabKey) for k, frame in pairs(contentFrames) do frame.Visible = (k == tabKey) end currentTab = tabKey end

-- Criar abas e conteúdo local index = 1 for icon, _ in pairs(tabs) do local button = createTabButton(icon, index) tabs[icon] = button

local frame = Instance.new("Frame", MainFrame)
frame.BackgroundTransparency = 1
frame.Position = UDim2.new(0, 60, 0, 10)
frame.Size = UDim2.new(1, -70, 1, -20)
frame.Visible = false
contentFrames[icon] = frame

button.MouseButton1Click:Connect(function()
    switchTab(icon)
end)

index += 1

end

-- HOME PAGE - "🏠" local homeText = Instance.new("TextLabel", contentFrames["🏠"]) homeText.Size = UDim2.new(1, 0, 0, 60) homeText.Position = UDim2.new(0, 0, 0, 0) homeText.Text = "Torill Hub criado por @bloxfruitisbetter\nDiscord: https://discord.gg/844JzWzC" homeText.TextColor3 = Color3.new(1, 1, 1) homeText.TextSize = 18 homeText.TextWrapped = true homeText.BackgroundTransparency = 1

-- FARM PAGE - "🗡️" local farmFrame = contentFrames["🗡️"]

-- Lista de mundos local WorldList = Instance.new("TextLabel", farmFrame) WorldList.Size = UDim2.new(0.4, -10, 1, -40) WorldList.Position = UDim2.new(0, 0, 0, 0) WorldList.TextXAlignment = Enum.TextXAlignment.Left WorldList.TextYAlignment = Enum.TextYAlignment.Top WorldList.BackgroundColor3 = Color3.fromRGB(40, 40, 40) WorldList.TextColor3 = Color3.new(1, 1, 1) WorldList.Text = "Mundos:\n" WorldList.TextWrapped = true WorldList.TextSize = 14

-- Lista de inimigos local EnemyList = Instance.new("TextLabel", farmFrame) EnemyList.Size = UDim2.new(0.6, -10, 1, -40) EnemyList.Position = UDim2.new(0.4, 10, 0, 0) EnemyList.TextXAlignment = Enum.TextXAlignment.Left EnemyList.TextYAlignment = Enum.TextYAlignment.Top EnemyList.BackgroundColor3 = Color3.fromRGB(40, 40, 40) EnemyList.TextColor3 = Color3.new(1, 1, 1) EnemyList.Text = "Inimigos:\n" EnemyList.TextWrapped = true EnemyList.TextSize = 14

-- Refresh botão local RefreshBtn = Instance.new("TextButton", farmFrame) RefreshBtn.Size = UDim2.new(0.3, 0, 0, 30) RefreshBtn.Position = UDim2.new(0.35, 0, 1, -30) RefreshBtn.Text = "🔄 Refresh Enemies" RefreshBtn.TextColor3 = Color3.new(1, 1, 1) RefreshBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)

RefreshBtn.MouseButton1Click:Connect(function() worldsList = getWorlds() if #worldsList > 0 then selectedWorld = worldsList[1] local enemies = getEnemies(selectedWorld) selectedEnemy = next(enemies)

WorldList.Text = "Mundos:\n" .. table.concat(worldsList, "\n")

    local enemyText = ""
    for k, _ in pairs(enemies) do
        enemyText = enemyText .. k .. "\n"
    end
    EnemyList.Text = "Inimigos:\n" .. enemyText
end

end)

-- Auto Farm toggle local AutoFarmToggle = Instance.new("TextButton", farmFrame) AutoFarmToggle.Size = UDim2.new(0.3, 0, 0, 30) AutoFarmToggle.Position = UDim2.new(0.05, 0, 1, -70) AutoFarmToggle.Text = "⚔️ Auto Farm [OFF]" AutoFarmToggle.TextColor3 = Color3.new(1, 1, 1) AutoFarmToggle.BackgroundColor3 = Color3.fromRGB(80, 80, 80)

AutoFarmToggle.MouseButton1Click:Connect(function() autoFarmEnabled = not autoFarmEnabled AutoFarmToggle.Text = autoFarmEnabled and "⚔️ Auto Farm [ON]" or "⚔️ Auto Farm [OFF]" if autoFarmEnabled then task.spawn(autoFarm) end end)

-- Auto Click toggle local AutoClickToggle = Instance.new("TextButton", farmFrame) AutoClickToggle.Size = UDim2.new(0.3, 0, 0, 30) AutoClickToggle.Position = UDim2.new(0.6, 0, 1, -70) AutoClickToggle.Text = "🖱️ Auto Click [OFF]" AutoClickToggle.TextColor3 = Color3.new(1, 1, 1) AutoClickToggle.BackgroundColor3 = Color3.fromRGB(80, 80, 80)

AutoClickToggle.MouseButton1Click:Connect(function() autoClickEnabled = not autoClickEnabled AutoClickToggle.Text = autoClickEnabled and "🖱️ Auto Click [ON]" or "🖱️ Auto Click [OFF]" if autoClickEnabled then task.spawn(autoClick) end end)

-- Inicializa na aba 🏠 switchTab("🏠")

