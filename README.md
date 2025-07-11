-- ✅ ESP + GUI + AIMBOT Toggle completo (sem botão direito)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Estados
local ESP_Ativo = false
local Aimbot_Ativo = false
local ESP_Boxes = {}

-- Criar GUI
local function CriarGUI()
    local gui = Instance.new("ScreenGui", game.CoreGui)
    gui.Name = "ESP_Aimbot_GUI"

    -- Botão ESP
    local espButton = Instance.new("TextButton", gui)
    espButton.Size = UDim2.new(0, 120, 0, 40)
    espButton.Position = UDim2.new(0, 20, 0, 60)
    espButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    espButton.TextColor3 = Color3.new(1, 1, 1)
    espButton.Font = Enum.Font.SourceSansBold
    espButton.TextSize = 20
    espButton.Text = "ESP: OFF"

    espButton.MouseButton1Click:Connect(function()
        ESP_Ativo = not ESP_Ativo
        espButton.Text = ESP_Ativo and "ESP: ON" or "ESP: OFF"
        for _, box in pairs(ESP_Boxes) do
            if box then box.Visible = false end
        end
    end)

    -- Botão AIMBOT
    local aimbotButton = Instance.new("TextButton", gui)
    aimbotButton.Size = UDim2.new(0, 180, 0, 40)
    aimbotButton.Position = UDim2.new(0, 20, 0, 110)
    aimbotButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    aimbotButton.TextColor3 = Color3.new(1, 1, 1)
    aimbotButton.Font = Enum.Font.SourceSansBold
    aimbotButton.TextSize = 20
    aimbotButton.Text = "Aimbot: OFF"

    aimbotButton.MouseButton1Click:Connect(function()
        Aimbot_Ativo = not Aimbot_Ativo
        aimbotButton.Text = Aimbot_Ativo and "Aimbot: ON" or "Aimbot: OFF"
    end)

    -- Info
    local infoText = Instance.new("TextLabel", gui)
    infoText.Size = UDim2.new(0, 300, 0, 30)
    infoText.Position = UDim2.new(0, 20, 0, 160)
    infoText.BackgroundTransparency = 1
    infoText.TextColor3 = Color3.new(1, 1, 1)
    infoText.Font = Enum.Font.SourceSans
    infoText.TextSize = 18
    infoText.Text = "Clique no botão para ativar o Aimbot"
end

-- Criar ESP box
local function CriarESP(player)
    if player == LocalPlayer or ESP_Boxes[player] then return end
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Thickness = 2
    box.Filled = false
    box.Transparency = 1
    ESP_Boxes[player] = box
end

local function RemoverESP(player)
    if ESP_Boxes[player] then
        ESP_Boxes[player]:Remove()
        ESP_Boxes[player] = nil
    end
end

-- Atualizar ESP
RunService.RenderStepped:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if player.Team ~= LocalPlayer.Team then
                local root = player.Character.HumanoidRootPart
                local pos, onScreen = Camera:WorldToViewportPoint(root.Position)

                if ESP_Ativo and onScreen then
                    CriarESP(player)
                    local box = ESP_Boxes[player]

                    local distance = (Camera.CFrame.Position - root.Position).Magnitude
                    local scale = 1 / distance * 100
                    local width = 30 * scale
                    local height = 60 * scale

                    box.Size = Vector2.new(width, height)
                    box.Position = Vector2.new(pos.X - width / 2, pos.Y - height / 2)
                    box.Visible = true
                elseif ESP_Boxes[player] then
                    ESP_Boxes[player].Visible = false
                end
            else
                if ESP_Boxes[player] then
                    ESP_Boxes[player].Visible = false
                end
            end
        else
            RemoverESP(player)
        end
    end

    for player in pairs(ESP_Boxes) do
        if not Players:FindFirstChild(player.Name) then
            RemoverESP(player)
        end
    end
end)

-- Função Aimbot
local function GetClosestPlayerToCrosshair()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            if player.Team ~= LocalPlayer.Team and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
                if onScreen then
                    local mousePos = UserInputService:GetMouseLocation()
                    local dist = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude

                    if dist < shortestDistance and dist < 300 then
                        shortestDistance = dist
                        closestPlayer = player
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- Atualizar aimbot
RunService.RenderStepped:Connect(function()
    if Aimbot_Ativo then
        local alvo = GetClosestPlayerToCrosshair()
        if alvo and alvo.Character and alvo.Character:FindFirstChild("Head") then
            local headPos = alvo.Character.Head.Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, headPos)
        end
    end
end)

-- Iniciar GUI
CriarGUI()
