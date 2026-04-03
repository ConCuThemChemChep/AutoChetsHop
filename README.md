-- [[ ULTIMATE CLEANER: WATERFALLS, CLOUDS & FOG ]] --
-- Optimizer by Phuc Admin

local workspace = game:GetService("Workspace")
local lighting = game:GetService("Lighting")

-- 1. Xóa sạch Mây và Sương mù (Atmosphere)
for _, obj in pairs(lighting:GetChildren()) do
    if obj:IsA("Clouds") or obj:IsA("Atmosphere") or obj:IsA("Sky") then
        obj:Destroy()
    end
end
lighting.FogEnd = 9e9 
lighting.FogStart = 9e9

-- 2. Truy quét và Xóa Thác nước & Hiệu ứng hạt
for _, v in pairs(workspace:GetDescendants()) do
    if v.Name:lower():find("waterfall") or v.Name:lower():find("water_fall") then
        v:Destroy()
    end
    if v:IsA("Beam") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Fire") or v:IsA("Smoke") then
        v:Destroy()
    end
    if v.Name:lower():find("foam") or v.Name:lower():find("bubble") then
        v:Destroy()
    end
end

-- 3. Tối ưu hóa bề mặt Nước
local terrain = workspace:FindFirstChildOfClass("Terrain")
if terrain then
    terrain.WaterWaveSize = 0
    terrain.WaterWaveSpeed = 0
    terrain.WaterReflectance = 0
    terrain.WaterTransparency = 1
end

-- [[ ETERNAL V31 - SUPER ANTI-KICK EDITION ]] --

if not game:IsLoaded() then game.Loaded:Wait() end

--// CẤU HÌNH HỆ THỐNG
_G.Config = {
    Enabled = true,
    FastMode = true,
    AntiLag = true,
    AntiAfk = true,
    StopOnRare = true, 
    FileName = "Phuc_ServerHistory.json",
    TargetServer = "#8372"
}

local LP = game.Players.LocalPlayer
local HTTP = game:GetService("HttpService")
local TS = game:GetService("TeleportService")
local CG = game:GetService("CoreGui")
local VU = game:GetService("VirtualUser")

--// ==========================================================
--// HỆ THỐNG ANTI-KICK & SERVER HOP SIÊU MƯỢT
--// ==========================================================

local function getVisitedHistory()
    if isfile(_G.Config.FileName) then
        local success, result = pcall(function() return HTTP:JSONDecode(readfile(_G.Config.FileName)) end)
        if success then return result end
    end
    return {}
end

local function saveToHistory(jobId)
    local history = getVisitedHistory()
    history[jobId] = os.time()
    for k, v in pairs(history) do
        if os.time() - v > 3600 then history[k] = nil end
    end
    writefile(_G.Config.FileName, HTTP:JSONEncode(history))
end

function HopServer()
    if _G.StatusLabel then _G.StatusLabel.Text = "🚀 ĐANG TÌM SERVER SIÊU SẠCH..." end
    
    local history = getVisitedHistory()
    local placeId = game.PlaceId
    
    -- API lấy server ưu tiên server vắng để farm rương nhanh
    local url = "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"
    
    local success, res = pcall(function() 
        return HTTP:JSONDecode(game:HttpGet(url)) 
    end)
    
    if success and res and res.data then
        for _, server in pairs(res.data) do
            if server.playing < server.maxPlayers 
               and server.id ~= game.JobId 
               and not history[server.id] 
               and server.id ~= _G.Config.TargetServer then
                
                saveToHistory(server.id)
                
                -- Bọc Teleport trong vòng lặp chống lỗi kết nối
                task.spawn(function()
                    while task.wait(0.5) do
                        TS:TeleportToPlaceInstance(placeId, server.id, LP)
                    end
                end)
                return
            end
        end
    end
    
    warn("🚨 Đang nhảy ngẫu nhiên để reset...")
    TS:Teleport(placeId)
end

--// HÀM KIỂM TRA ITEM HIẾM
local function HasRareItem()
    local RareItems = {"Fist of Darkness", "God's Chalice"}
    for _, item in pairs(LP.Backpack:GetChildren()) do
        if table.find(RareItems, item.Name) then return true end
    end
    if LP.Character then
        for _, item in pairs(LP.Character:GetChildren()) do
            if table.find(RareItems, item.Name) then return true end
        end
    end
    return false
end

--// BẢN ANTI-KICK MỚI (TỰ ĐỘNG NHẢY KHI CÓ THÔNG BÁO LỖI)
local function ApplyAntiKick()
    -- Chống AFK 20 phút
    LP.Idled:Connect(function()
        VU:CaptureController()
        VU:ClickButton2(Vector2.new())
    end)

    -- Bypass Error Prompts (Khi bị Kick hoặc Server Shutdown)
    local guiService = game:GetService("GuiService")
    guiService.ErrorMessageChanged:Connect(function()
        local msg = guiService:GetErrorMessage()
        if msg ~= "" then
            print("🚨 Phát hiện lỗi: " .. msg .. " -> Đang cứu hộ!")
            HopServer()
        end
    end)

    -- Phản hồi CoreGui
    CG.RobloxPromptGui.promptOverlay.ChildAdded:Connect(function(child)
        if child.Name == "ErrorPrompt" then 
            HopServer() 
        end
    end)
end

--// TỐI ƯU ĐỒ HỌA TUYỆT ĐỐI
local function SuperOptimize()
    if _G.Config.AntiLag then
        settings().Rendering.QualityLevel = 1
        for _, v in pairs(game:GetDescendants()) do
            if v:IsA("BasePart") then
                v.Material = Enum.Material.SmoothPlastic
                v.CastShadow = false
            elseif v:IsA("Decal") or v:IsA("Texture") or v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v:Destroy()
            end
        end
        game:GetService("Lighting").GlobalShadows = false
    end
end

--// QUÉT RƯƠNG (SPEED X10)
local function ScanChests()
    local target, shortestDist, chestCount = nil, math.huge, 0
    local char = LP.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return nil, 0 end
    
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("TouchTransmitter") and v.Parent and v.Parent.Name:lower():find("chest") then
            chestCount = chestCount + 1
            local dist = (char.HumanoidRootPart.Position - v.Parent.Position).Magnitude
            if dist < shortestDist then
                shortestDist = dist
                target = v.Parent
            end
        end
    end
    return target, chestCount
end

--// GIAO DIỆN & MOBILE BUTTON
if CG:FindFirstChild("EternalUI") then CG.EternalUI:Destroy() end
local ui = Instance.new("ScreenGui", CG); ui.Name = "EternalUI"

local openBtn = Instance.new("TextButton", ui)
openBtn.Size = UDim2.new(0, 50, 0, 50); openBtn.Position = UDim2.new(0, 10, 0.5, -25)
openBtn.Text = "E"; openBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 255)
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(1, 0)

local main = Instance.new("Frame", ui)
main.Size = UDim2.new(0, 250, 0, 200); main.Position = UDim2.new(0.5, -125, 0.3, 0)
main.BackgroundColor3 = Color3.fromRGB(10, 10, 10); main.Visible = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", main); stroke.Color = Color3.fromRGB(0, 255, 255)

openBtn.MouseButton1Click:Connect(function() main.Visible = not main.Visible end)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 40); title.Text = "ETERNAL V31 PHUC"; title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold; title.BackgroundTransparency = 1

local toggleBtn = Instance.new("TextButton", main)
toggleBtn.Size = UDim2.new(0.85, 0, 0, 45); toggleBtn.Position = UDim2.new(0.075, 0, 0.25, 0)
toggleBtn.Text = _G.Config.Enabled and "AUTO CHEST: ON" or "AUTO CHEST: OFF"
toggleBtn.BackgroundColor3 = _G.Config.Enabled and Color3.fromRGB(0, 120, 60) or Color3.fromRGB(30, 30, 30)
toggleBtn.TextColor3 = Color3.new(1, 1, 1); toggleBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", toggleBtn)

local st = Instance.new("TextLabel", main)
st.Size = UDim2.new(1, 0, 0, 30); st.Position = UDim2.new(0, 0, 0.55, 0)
st.Text = "Đang khởi tạo..."; st.TextColor3 = Color3.fromRGB(150, 150, 150)
st.Font = Enum.Font.Code; st.BackgroundTransparency = 1; _G.StatusLabel = st

toggleBtn.MouseButton1Click:Connect(function()
    _G.Config.Enabled = not _G.Config.Enabled
    toggleBtn.Text = _G.Config.Enabled and "AUTO CHEST: ON" or "AUTO CHEST: OFF"
    toggleBtn.BackgroundColor3 = _G.Config.Enabled and Color3.fromRGB(0, 120, 60) or Color3.fromRGB(30, 30, 30)
end)

--// VẬN HÀNH (CORE LOGIC)
task.spawn(function()
    while task.wait() do
        if _G.Config.Enabled then
            if _G.Config.StopOnRare and HasRareItem() then
                _G.Config.Enabled = false
                _G.StatusLabel.Text = "🎁 CÓ ĐỒ HIẾM! ĐÃ DỪNG."
                continue
            end

            local chest, total = ScanChests()
            local char = LP.Character
            if chest and total > 0 and char and char:FindFirstChild("HumanoidRootPart") then
                _G.StatusLabel.Text = "💎 RƯƠNG CÒN: " .. total
                char.HumanoidRootPart.CFrame = chest.CFrame
                firetouchinterest(char.HumanoidRootPart, chest, 0)
                firetouchinterest(char.HumanoidRootPart, chest, 1)
                -- FastMode cực nhanh cho server mới reset
                if _G.Config.FastMode then task.wait(0.01) end
            else
                _G.StatusLabel.Text = "🚨 HẾT RƯƠNG -> ĐỔI SERVER..."
                task.wait(0.5)
                HopServer()
                break
            end
        end
    end
end)

SuperOptimize()
ApplyAntiKick()
print("--- [ETERNAL V31 LOADED: SIÊU MƯỢT] ---")
