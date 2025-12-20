--// Caio_hub • Reset + Block Respawn 8s + 25 Clone Desync
--// LocalScript / Executor

local Players = game:GetService("Players")
local lp = Players.LocalPlayer

local BLOCK_TIME = 8
local TOTAL_CLONES = 25
local busy = false
local blockConn
local blockRespawn = false

--================ GUI =================
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "Caio_hub_OneButton"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,300,0,150)
frame.Position = UDim2.new(0.5,-150,0.45,0)
frame.BackgroundColor3 = Color3.fromRGB(128,0,128) -- Roxa
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,14)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "Caio_hub • DESYNC"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255,255,255)

local btn = Instance.new("TextButton", frame)
btn.Size = UDim2.new(0.85,0,0,50)
btn.Position = UDim2.new(0.075,0,0.5,0)
btn.BackgroundColor3 = Color3.fromRGB(50,0,50) -- Tom de roxo mais escuro
btn.Text = "Desync"
btn.Font = Enum.Font.GothamBold
btn.TextSize = 16
btn.TextColor3 = Color3.fromRGB(255,255,255)
Instance.new("UICorner", btn).CornerRadius = UDim.new(0,12)

--================ CLONE =================
local function createClone(char, offset)
    local clone = char:Clone()
    clone.Name = "DESYNC_CLONE"
    clone.Parent = workspace

    for _,v in ipairs(clone:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Anchored = true
            v.CanCollide = false
            v.CFrame = v.CFrame + offset
        elseif v:IsA("Humanoid") then
            v.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
        end
    end
end

--================ BOTÃO =================
btn.MouseButton1Click:Connect(function()
    if busy then return end
    busy = true

    local char = lp.Character
    if not char then busy = false return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then busy = false return end

    -- BLOQUEIA RESPAWN AUTOMÁTICO
    blockRespawn = true
    if blockConn then blockConn:Disconnect() end
    blockConn = lp.CharacterAdded:Connect(function(c)
        if blockRespawn then
            task.wait()
            c:Destroy()
        end
    end)

    -- MATA e DESTRÓI o personagem
    hum.Health = 0
    task.wait(0.1)
    if lp.Character then
        lp.Character:Destroy()
    end

    -- CONTAGEM REGRESSIVA
    for i = BLOCK_TIME, 0, -1 do
        btn.Text = "Respawn em: "..i
        task.wait(1)
    end

    -- LIBERA RESPAWN AUTOMÁTICO
    blockRespawn = false
    if blockConn then
        blockConn:Disconnect()
        blockConn = nil
    end
    btn.Text = "Desync"

    -- FORÇA RESPAWN SEGURO
    lp:LoadCharacter()
    local newChar = lp.CharacterAdded:Wait()
    task.wait(0.2)

    -- 1 CLONE INICIAL
    createClone(newChar, Vector3.new(0,0,0))

    -- 25 CLONES NO RESPAWN
    for i = 1, TOTAL_CLONES do
        local offset = Vector3.new(math.random(-6,6),0,math.random(-6,6))
        createClone(newChar, offset)
        task.wait(0.03)
    end

    busy = false
end)
