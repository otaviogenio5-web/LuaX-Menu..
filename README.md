local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local lp = Players.LocalPlayer

-- Configurações e Remotes
local remoteBixo = ReplicatedStorage:WaitForChild("RemoteNovos"):WaitForChild("bixobrabo")
local invRequest = ReplicatedStorage:WaitForChild("Modules"):WaitForChild("InvRemotes"):WaitForChild("InvRequest")
local itensRoubo = {"AK47", "Uzi", "Glock17", "Glock 17", "Parafal", "PARAFAL", "Faca", "IA2", "G3", "Tratamento", "FACA", "Xbox", "Hi Power", "Natalina", "C4", "AR-15", "AR15", "Escudo", "ESCUDO"}

-- Estados Globais
_G.AutoRoubarAtivo = false
_G.RevistarToggle = false
_G.ESP_Ativo = false
_G.AutoCL_Ativo = false

-- Variáveis do FLY
local flySpeed = 240
local fallSpeed = -6
local flying = false
local flyConnection

-- GUI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", lp.PlayerGui)
ScreenGui.Name = "ExperienceGui_System"
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 420, 0, 350) -- Aumentei um pouco para caber tudo
MainFrame.Position = UDim2.new(0.5, -210, 0.5, -175)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 15)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

-- BARRA LATERAL
local SideBar = Instance.new("Frame", MainFrame)
SideBar.Size = UDim2.new(0, 100, 1, 0)
SideBar.BackgroundColor3 = Color3.fromRGB(8, 8, 10)
Instance.new("UICorner", SideBar).CornerRadius = UDim.new(0, 12)

-- CONTAINER DE PÁGINAS
local Content = Instance.new("Frame", MainFrame)
Content.Position = UDim2.new(0, 110, 0, 10)
Content.Size = UDim2.new(1, -120, 1, -20)
Content.BackgroundTransparency = 1

local function createPage()
    local f = Instance.new("Frame", Content)
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundTransparency = 1
    f.Visible = false
    return f
end

local PageFarm = createPage()
local PageInv = createPage()
local PageESP = createPage()
local PageFly = createPage()
local PageOutros = createPage()
PageFarm.Visible = true

-- BOTÕES DAS ABAS
local function createTabBtn(name, pos, page)
    local btn = Instance.new("TextButton", SideBar)
    btn.Size = UDim2.new(1, -10, 0, 40)
    btn.Position = UDim2.new(0, 5, 0, pos)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(function()
        PageFarm.Visible = false PageInv.Visible = false PageESP.Visible = false PageFly.Visible = false PageOutros.Visible = false
        page.Visible = true
    end)
end

createTabBtn("FARM", 10, PageFarm)
createTabBtn("INV", 55, PageInv)
createTabBtn("VISUAL", 100, PageESP)
createTabBtn("FLY/MOV", 145, PageFly)
createTabBtn("OUTROS", 190, PageOutros)

local function criarBotao(txt, parent, pos, color)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 45)
    btn.Position = UDim2.new(0, 0, 0, pos)
    btn.Text = txt
    btn.BackgroundColor3 = color or Color3.fromRGB(30, 30, 35)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    Instance.new("UICorner", btn)
    return btn
end

-- ==========================================
-- LÓGICA FARM: AUTO ROUBAR, REVISTAR E BAÚ
-- ==========================================
local bRoubar = criarBotao("AUTO ROUBAR: OFF", PageFarm, 0)
bRoubar.MouseButton1Click:Connect(function()
    _G.AutoRoubarAtivo = not _G.AutoRoubarAtivo
    bRoubar.Text = "AUTO ROUBAR: " .. (_G.AutoRoubarAtivo and "ON" or "OFF")
    bRoubar.BackgroundColor3 = _G.AutoRoubarAtivo and Color3.fromRGB(255, 40, 120) or Color3.fromRGB(30, 30, 35)
end)

local bRev = criarBotao("Auto Revistar: OFF", PageFarm, 55)
bRev.MouseButton1Click:Connect(function()
    _G.RevistarToggle = not _G.RevistarToggle
    bRev.Text = "Auto Revistar: " .. (_G.RevistarToggle and "ON" or "OFF")
    bRev.BackgroundColor3 = _G.RevistarToggle and Color3.fromRGB(122, 28, 187) or Color3.fromRGB(30, 30, 35)
end)

-- FUNÇÃO ABRIR BAÚ (LUAX HUB INTEGRADA)
local bBau = criarBotao("🎁 ABRIR BAÚ", PageFarm, 110, Color3.fromRGB(180, 120, 0))
bBau.MouseButton1Click:Connect(function()
    bBau.Text = "Abrindo..."
    local guardarArgs = {"trasnferebau", "Entro", "Tratamento", 5, 1}
    task.spawn(function()
        invRequest:InvokeServer(unpack(guardarArgs))
    end)
    task.wait(0.5)
    bBau.Text = "Baú Aberto!"
    task.wait(1)
    bBau.Text = "🎁 ABRIR BAÚ"
end)

-- ==========================================
-- ESP VIP (TRAZIDO DE VOLTA)
-- ==========================================
local espData = {}
local function createEspForPlayer(player)
    if player == lp then return end
    local elements = {}
    local label = Instance.new("TextLabel", ScreenGui)
    label.BackgroundTransparency = 1; label.TextColor3 = Color3.new(1,1,1); label.TextStrokeTransparency = 0.2
    label.Size = UDim2.new(0, 150, 0, 40); label.TextScaled = true; label.Visible = false
    elements.Label = label
    local line = Instance.new("Frame", ScreenGui)
    line.BorderSizePixel = 0; line.BackgroundColor3 = Color3.new(1,1,1); line.AnchorPoint = Vector2.new(0.5,0.5); line.Visible = false
    local grad = Instance.new("UIGradient", line)
    grad.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.new(1,0,0)), ColorSequenceKeypoint.new(1, Color3.new(1,1,1))})
    grad.Rotation = 90
    elements.Line = line
    espData[player] = elements
    return elements
end

local bESP = criarBotao("ESP VIP: OFF", PageESP, 0)
bESP.MouseButton1Click:Connect(function()
    _G.ESP_Ativo = not _G.ESP_Ativo
    bESP.Text = "ESP VIP: " .. (_G.ESP_Ativo and "ON" or "OFF")
    bESP.BackgroundColor3 = _G.ESP_Ativo and Color3.fromRGB(122, 28, 187) or Color3.fromRGB(30, 30, 35)
end)

-- ==========================================
-- LOOPS E SISTEMAS (FLY, CL, ROUBO)
-- ==========================================
task.spawn(function()
    while true do
        if _G.AutoRoubarAtivo then
            for _, gui in ipairs(lp.PlayerGui:GetChildren()) do if gui.Name == "NotifyGui" then gui:Destroy() end end
            local args = {[1] = "mudaInv", [4] = "1"}
            for i, item in ipairs(itensRoubo) do
                if i <= 16 then
                    args[3] = item; args[2] = tostring(i)
                    task.spawn(function() invRequest:InvokeServer(unpack(args)) end)
                end
            end
        end
        task.wait(0)
    end
end)

RunService.RenderStepped:Connect(function()
    -- ESP VIP RENDER
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= lp and player.Character then
            local data = espData[player]
            if _G.ESP_Ativo then
                if not data then data = createEspForPlayer(player) end
                local head = player.Character:FindFirstChild("Head")
                if head and data then
                    local headPos, onScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 2, 0))
                    if onScreen then
                        local tool = player.Character:FindFirstChildOfClass("Tool")
                        data.Label.Text = string.format("%s\nMão: %s", player.Name, tool and tool.Name or "Vazia")
                        data.Label.Position = UDim2.new(0, headPos.X - 75, 0, headPos.Y - 50)
                        data.Label.Visible = true
                        local startPos = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                        local endPos = Vector2.new(headPos.X, headPos.Y)
                        data.Line.Size = UDim2.new(0, 2, 0, (startPos - endPos).Magnitude)
                        data.Line.Position = UDim2.new(0, (startPos.X + endPos.X) / 2, 0, (startPos.Y + endPos.Y) / 2)
                        data.Line.Rotation = math.deg(math.atan2(endPos.Y - startPos.Y, endPos.X - startPos.X)) - 90
                        data.Line.Visible = true
                    else data.Label.Visible = false data.Line.Visible = false end
                end
            elseif data then
                data.Label.Visible = false; data.Line.Visible = false
            end
        end
    end
end)

-- Botão Fly e Auto CL (Páginas Fly e Outros)
local flyBtn = criarBotao("Fly: OFF", PageFly, 0)
flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    flyBtn.Text = flying and "Fly: ON" or "Fly: OFF"
    if flying then
        startFly = RunService.Heartbeat:Connect(function()
            if not flying then startFly:Disconnect() return end
            local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + Camera.CFrame.LookVector)
                hrp.Velocity = (lp.Character.Humanoid.MoveDirection.Magnitude > 0) and (Camera.CFrame.LookVector * flySpeed) or Vector3.new(0, fallSpeed, 0)
                for _,v in ipairs(lp.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end
            end
        end)
    end
end)

local bCL = criarBotao("Auto CL: OFF", PageOutros, 0)
bCL.MouseButton1Click:Connect(function()
    _G.AutoCL_Ativo = not _G.AutoCL_Ativo
    bCL.Text = "Auto CL: " .. (_G.AutoCL_Ativo and "ON" or "OFF")
    bCL.BackgroundColor3 = _G.AutoCL_Ativo and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(30, 30, 35)
end)

lp.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid").HealthChanged:Connect(function(h) if _G.AutoCL_Ativo and h <= 0 then lp:Kick("AUTO CL") end end)
end)

-- Botão M lateral
local MBtn = Instance.new("TextButton", ScreenGui)
MBtn.Size = UDim2.new(0, 45, 0, 45); MBtn.Position = UDim2.new(0, 10, 0.5, -22)
MBtn.Text = "M"; MBtn.BackgroundColor3 = Color3.fromRGB(122, 28, 187); MBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", MBtn)
MBtn.MouseButton1Click:Connect(function() MainFrame.Visible = not MainFrame.Visible end)
