-- [[ NEXUS MASTER V12 - LEGIT AIM & STABLE ]] --
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

if CoreGui:FindFirstChild("NEXUS_V12") then CoreGui:FindFirstChild("NEXUS_V12"):Destroy() end

local flags = {
    aim = false,
    esp = false,
    hitbox = false,
    bhop = false,
    wallCheck = false,
    teamCheck = false,
    fovRadius = 150,
    hitboxValue = 10,
    bhopBoost = 8,
    currentSpeed = 16
}

-- Визуальный круг FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1.5
fovCircle.Color = Color3.fromRGB(191, 64, 255)
fovCircle.Visible = false

-- UI
local ScreenGui = Instance.new("ScreenGui", CoreGui); ScreenGui.Name = "NEXUS_V12"
local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 300, 0, 480); Main.Position = UDim2.new(0.5, -150, 0.5, -240)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 17); Main.BorderSizePixel = 0
Instance.new("UICorner", Main)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(50, 50, 55)

-- Функция перетаскивания (Draggable)
local d, di, ds, sp
Main.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then d = true; ds = i.Position; sp = Main.Position end end)
Main.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement then di = i end end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then d = false end end)
RunService.RenderStepped:Connect(function() if d and di then
    local del = di.Position - ds
    Main.Position = UDim2.new(sp.X.Scale, sp.X.Offset + del.X, sp.Y.Scale, sp.Y.Offset + del.Y)
end end)

local Container = Instance.new("ScrollingFrame", Main); Container.Size = UDim2.new(1, -20, 1, -60); Container.Position = UDim2.new(0, 10, 0, 50); Container.BackgroundTransparency = 1; Container.ScrollBarThickness = 0
Instance.new("UIListLayout", Container).Padding = UDim.new(0, 8)

-- Конструкторы кнопок и ползунков
local function CreateToggle(name, var)
    local b = Instance.new("TextButton", Container); b.Size = UDim2.new(1, 0, 0, 35); b.Text = name; b.BackgroundColor3 = Color3.fromRGB(30, 30, 33); b.TextColor3 = Color3.new(0.7,0.7,0.7); b.Font = Enum.Font.GothamMedium; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(function() flags[var] = not flags[var]; b.BackgroundColor3 = flags[var] and Color3.fromRGB(191, 64, 255) or Color3.fromRGB(30, 30, 33); b.TextColor3 = flags[var] and Color3.new(1,1,1) or Color3.new(0.7,0.7,0.7) end)
end

local function CreateSlider(name, min, max, var)
    local f = Instance.new("Frame", Container); f.Size = UDim2.new(1, 0, 0, 45); f.BackgroundTransparency = 1
    local l = Instance.new("TextLabel", f); l.Size = UDim2.new(1, 0, 0, 15); l.Text = name .. ": " .. flags[var]; l.TextColor3 = Color3.new(0.8,0.8,0.8); l.BackgroundTransparency = 1; l.Font = Enum.Font.Gotham; l.TextSize = 12; l.TextXAlignment = Enum.TextXAlignment.Left
    local bar = Instance.new("Frame", f); bar.Size = UDim2.new(1, 0, 0, 4); bar.Position = UDim2.new(0, 0, 0, 25); bar.BackgroundColor3 = Color3.fromRGB(45,45,48)
    local fill = Instance.new("Frame", bar); fill.Size = UDim2.new((flags[var]-min)/(max-min), 0, 1, 0); fill.BackgroundColor3 = Color3.fromRGB(191,64,255)
    bar.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then local conn; conn = UserInputService.InputChanged:Connect(function(i2) if i2.UserInputType == Enum.UserInputType.MouseMovement then
        local p = math.clamp((i2.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1); fill.Size = UDim2.new(p, 0, 1, 0); local v = math.floor(min + (max - min) * p); flags[var] = v; l.Text = name .. ": " .. v end end)
        UserInputService.InputEnded:Connect(function(i3) if i3.UserInputType == Enum.UserInputType.MouseButton1 then conn:Disconnect() end end) end end)
end

CreateToggle("AIMBOT (Hold RMB)", "aim")
CreateSlider("FOV Radius", 50, 800, "fovRadius")
CreateToggle("Hitbox Expander", "hitbox")
CreateSlider("Hitbox Size", 2, 30, "hitboxValue")
CreateToggle("Master ESP", "esp")
CreateToggle("Wall Check", "wallCheck")
CreateToggle("Team Check", "teamCheck")
CreateToggle("Infinite Bhop", "bhop")
CreateSlider("Bhop Boost", 1, 50, "bhopBoost")

-- Поиск цели для обычного Аимбота
local function getTarget()
    local t, d = nil, flags.fovRadius
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            if not flags.teamCheck or p.Team ~= player.Team then
                local part = p.Character:FindFirstChild("Head") or p.Character:FindFirstChild("HumanoidRootPart")
                if part then
                    local pos, vis = camera:WorldToViewportPoint(part.Position)
                    if vis then
                        local mD = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                        if mD < d then
                            local blocked = false
                            if flags.wallCheck then
                                local ray = RaycastParams.new(); ray.FilterDescendantsInstances = {player.Character, camera}; ray.FilterType = Enum.RaycastFilterType.Exclude
                                local res = workspace:Raycast(camera.CFrame.Position, (part.Position - camera.CFrame.Position).Unit * 1000, ray)
                                if not (res and res.Instance:IsDescendantOf(p.Character)) then blocked = true end
                            end
                            if not blocked then t = part; d = mD end
                        end
                    end
                end
            end
        end
    end
    return t
end

-- Цикл
RunService.RenderStepped:Connect(function()
    fovCircle.Visible = flags.aim
    fovCircle.Radius = flags.fovRadius
    fovCircle.Position = UserInputService:GetMouseLocation()

    -- AIMBOT (Доводка камеры)
    if flags.aim and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getTarget()
        if target then camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position) end
    end

    -- Остальные функции
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            local h = p.Character:FindFirstChild("Head")
            if h then
                h.Size = flags.hitbox and Vector3.new(flags.hitboxValue, flags.hitboxValue, flags.hitboxValue) or Vector3.new(2,1,1)
                h.Transparency = flags.hitbox and 0.6 or 0
                h.CanCollide = false
            end
            if flags.esp and not p.Character:FindFirstChild("NexusHL") then
                local hl = Instance.new("Highlight", p.Character); hl.Name = "NexusHL"; hl.FillColor = Color3.fromRGB(191, 64, 255)
            elseif not flags.esp and p.Character:FindFirstChild("NexusHL") then p.Character.NexusHL:Destroy() end
        end
    end

    local char = player.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChild("Humanoid")
    if flags.bhop and hum and hrp and UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        if hum.FloorMaterial ~= Enum.Material.Air then hum:ChangeState(Enum.HumanoidStateType.Jumping); flags.currentSpeed = flags.currentSpeed + flags.bhopBoost end
        hrp.Velocity = Vector3.new(hum.MoveDirection.X * flags.currentSpeed, hrp.Velocity.Y, hum.MoveDirection.Z * flags.currentSpeed)
    else flags.currentSpeed = 16 end
end)
