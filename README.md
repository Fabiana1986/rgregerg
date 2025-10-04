-- LucaxxxHub Rayfield Panel (com FOV dinâmico e funcional, ESP box branco)

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- FOV Vars
getgenv().lucaxx_fov_enabled = false
getgenv().lucaxx_fov_size = 240 -- Padrão
local fovDrawing = nil

local function updateFOV()
    if not getgenv().lucaxx_fov_enabled then
        if fovDrawing then
            fovDrawing.Visible = false
        end
        return
    end
    if not fovDrawing then
        fovDrawing = Drawing.new("Circle")
        fovDrawing.Color = Color3.fromRGB(255,0,0)
        fovDrawing.Thickness = 2
        fovDrawing.Filled = false
        fovDrawing.Transparency = 0.7
    end
    fovDrawing.Radius = getgenv().lucaxx_fov_size/2
    fovDrawing.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    fovDrawing.Visible = true
end

RunService.RenderStepped:Connect(function()
    if fovDrawing and getgenv().lucaxx_fov_enabled then
        fovDrawing.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        fovDrawing.Radius = getgenv().lucaxx_fov_size/2
        fovDrawing.Visible = true
    end
end)

-- ESP Table
local espObjects = {}

local function clearESP()
    for _,adorns in pairs(espObjects) do
        for _,ad in pairs(adorns) do
            if ad and ad.Parent then
                ad:Destroy()
            end
        end
    end
    espObjects = {}
end

local function createESP(plr)
    if plr == LocalPlayer then return end
    if not plr.Character then return end
    espObjects[plr] = {}
    for _,part in ipairs(plr.Character:GetDescendants()) do
        if part:IsA("BasePart") then
            local adorn = Instance.new("BoxHandleAdornment")
            adorn.Adornee = part
            adorn.AlwaysOnTop = true
            adorn.ZIndex = 10
            adorn.Size = part.Size
            adorn.Color3 = Color3.fromRGB(255,255,255)
            adorn.Transparency = 0.6
            adorn.Parent = LocalPlayer.PlayerGui
            table.insert(espObjects[plr], adorn)
        end
    end
end

local function updateESP()
    clearESP()
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            createESP(plr)
        end
    end
end

local espConnection
local renderESPConnection
local function toggleESP(state)
    if state then
        updateESP()
        espConnection = Players.PlayerAdded:Connect(function(plr)
            plr.CharacterAdded:Connect(function()
                wait(1)
                updateESP()
            end)
        end)
        renderESPConnection = RunService.RenderStepped:Connect(updateESP)
    else
        if espConnection then espConnection:Disconnect() espConnection = nil end
        if renderESPConnection then renderESPConnection:Disconnect() renderESPConnection = nil end
        clearESP()
    end
end

-- AIMBOT
local aimConnection
local function getClosestPlayerToMouse()
    local closestPlayer
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer
end

local function startAimbot()
    if aimConnection then aimConnection:Disconnect() end
    aimConnection = RunService.RenderStepped:Connect(function()
        local target = getClosestPlayerToMouse()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, headPos)
        end
    end)
end

local function stopAimbot()
    if aimConnection then
        aimConnection:Disconnect()
        aimConnection = nil
    end
end

local function toggleAimbot(state)
    if state then
        startAimbot()
    else
        stopAimbot()
    end
end

-- Speed
local origWalkSpeed = 16
local function toggleSpeed(state)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        if state then
            origWalkSpeed = LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 100
        else
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = origWalkSpeed
        end
    end
end

-- AimKill
local function toggleAimKill(state)
    if state then
        for _,plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChildOfClass("Humanoid") then
                plr.Character:FindFirstChildOfClass("Humanoid").Health = 0
            end
        end
    end
end

-- Infinite Jump
local infiniteJumpConnection
local function toggleInfJump(state)
    if state then
        if infiniteJumpConnection then infiniteJumpConnection:Disconnect() end
        infiniteJumpConnection = UserInputService.JumpRequest:Connect(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        if infiniteJumpConnection then infiniteJumpConnection:Disconnect() infiniteJumpConnection = nil end
    end
end

-- Infinite Life
local infLifeConnection
local function toggleInfLife(state)
    if state then
        if infLifeConnection then infLifeConnection:Disconnect() end
        infLifeConnection = RunService.Heartbeat:Connect(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if hum.Health < hum.MaxHealth then
                    hum.Health = hum.MaxHealth
                end
            end
        end)
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            hum.Health = hum.MaxHealth
        end
    else
        if infLifeConnection then infLifeConnection:Disconnect() infLifeConnection = nil end
    end
end

-- ------------------- Rayfield Menu ------------------- --
local Window = Rayfield:CreateWindow({
    Name = "LucaxxxHub Rayfield Panel",
    LoadingTitle = "LucaxxxHub Panel",
    LoadingSubtitle = "by Lucaxxx",
    ConfigurationSaving = {
        Enabled = false,
    },
    KeySystem = true,
    KeySettings = {
        Title = "LucaxxxHub Key",
        Subtitle = "Digite a chave para acessar",
        Note = "Key: lucaxx",
        FileName = "lucaxxhub_key.txt",
        SaveKey = false,
        GrabKeyFromSite = false,
        Key = {"lucaxx"}
    }
})

local MainTab = Window:CreateTab("Painel", 4483362458)

MainTab:CreateToggle({
    Name = "Aimbot (Gruda na cabeça)",
    CurrentValue = false,
    Callback = function(Value)
        toggleAimbot(Value)
    end
})

MainTab:CreateToggle({
    Name = "ESP Box Branco",
    CurrentValue = false,
    Callback = function(Value)
        toggleESP(Value)
    end
})

MainTab:CreateToggle({
    Name = "FOV (círculo vermelho)",
    CurrentValue = false,
    Callback = function(Value)
        getgenv().lucaxx_fov_enabled = Value
        updateFOV()
    end
})

MainTab:CreateSlider({
    Name = "Tamanho do FOV",
    Range = {50, 600},
    Increment = 1,
    CurrentValue = getgenv().lucaxx_fov_size,
    Callback = function(Value)
        getgenv().lucaxx_fov_size = Value
        updateFOV()
    end
})

MainTab:CreateToggle({
    Name = "Speed (100)",
    CurrentValue = false,
    Callback = function(Value)
        toggleSpeed(Value)
    end
})

MainTab:CreateButton({
    Name = "AimKill (Mata todos)",
    Callback = function()
        toggleAimKill(true)
    end
})

MainTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = function(Value)
        toggleInfJump(Value)
    end
})

MainTab:CreateToggle({
    Name = "Infinite Life",
    CurrentValue = false,
    Callback = function(Value)
        toggleInfLife(Value)
    end
})

Rayfield:Notify({
    Title = "LucaxxxHub Rayfield",
    Content = "Script carregado! Use a key: lucaxx",
    Duration = 5
})
