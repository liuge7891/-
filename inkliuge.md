-- ============================================================
--  墨水脚本 | Ink Hub 完整版
-- ============================================================

local GlobalEnv = getgenv()
local FunctionEnv = getfenv()

-- UI 库
local repo = 'https://raw.githubusercontent.com/DevSloPo/obsidian_UI/main/'
local Library = loadstring(game:HttpGet(repo .. 'Library.lua'))()
local ThemeManager = loadstring(game:HttpGet(repo .. 'addons/ThemeManager.lua'))()
local SaveManager = loadstring(game:HttpGet(repo .. 'addons/SaveManager.lua'))()

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Options = Library.Options
local Toggles = Library.Toggles

-- 创建窗口（已去掉图片）
local Window = Library:CreateWindow({
    AutoShow = true,
    Title = "墨水脚本",
    NotifySide = "Right",
    ShowCustomCursor = true,
})

-- 标签页
local RedLightTab       = Window:AddTab("木头人", "lightbulb")
local DalgonaTab        = Window:AddTab("糖饼", "cake")
local TugOfWarTab       = Window:AddTab("拔河", "sports_kabaddi")
local HideAndSeekTab    = Window:AddTab("捉迷藏", "visibility")
local JumpRopeTab       = Window:AddTab("跳绳/玻璃桥", "sports_handball")
local MingleTab         = Window:AddTab("混合/旋转木马", "group")
local LastSupperTab     = Window:AddTab("最后的晚餐", "restaurant")
local VillainTab        = Window:AddTab("反派", "dangerous")
local RandomFeaturesTab = Window:AddTab("随机功能", "casino")
local RebelTab          = Window:AddTab("反抗", "swords")
local FinalTab          = Window:AddTab("天空/最终", "sports_kabaddi")
local UISettingsTab     = Window:AddTab("界面设置", "settings")

-- ==========================================================
-- 通用工具函数
-- ==========================================================
local function GetRoot()
    local char = LocalPlayer.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function GetHumanoid()
    local char = LocalPlayer.Character
    return char and char:FindFirstChildOfClass("Humanoid")
end

local function Notify(text, duration)
    Library:Notify(text, duration or 3)
end

-- 通用：在多个路径下寻找远程事件
local function FindRemote(names, container)
    container = container or ReplicatedStorage
    local searchRoots = {
        container,
        container:FindFirstChild("Remotes"),
        container:FindFirstChild("RemoteEvents"),
        container:FindFirstChild("RemoteFunctions"),
    }
    for _, root in ipairs(searchRoots) do
        if root then
            for _, name in ipairs(names) do
                local obj = root:FindFirstChild(name)
                if obj and (obj:IsA("RemoteEvent") or obj:IsA("BindableEvent")) then
                    return obj
                end
            end
        end
    end
    return nil
end

-- 通用：生成防坠落平台
local function CreatePlatform(name, pos, size, color)
    local old = Workspace:FindFirstChild(name)
    if old then old:Destroy() end
    local p = Instance.new("Part")
    p.Name = name
    p.Size = size or Vector3.new(100, 1, 100)
    p.Anchored = true
    p.CanCollide = true
    p.Position = pos
    p.Material = Enum.Material.SmoothPlastic
    p.Color = color or Color3.fromRGB(120, 120, 120)
    p.Transparency = 0.3
    p.Parent = Workspace
    return p
end

-- ==========================================================
-- 木头人
-- ==========================================================
local RedLightMainBox = RedLightTab:AddLeftGroupbox("主要功能")
local RedLightUtilBox = RedLightTab:AddRightGroupbox("实用工具")

RedLightMainBox:AddButton("传送到终点", function()
    local hrp = GetRoot()
    if hrp then
        hrp.CFrame = CFrame.new(Vector3.new(-48.58, 1148.54, 197.73))
        Notify("已传送到木头人终点")
    end
end)

RedLightMainBox:AddButton("传送到起点", function()
    local hrp = GetRoot()
    if hrp then
        hrp.CFrame = CFrame.new(Vector3.new(-47.47, 1024.51, -566.99))
        Notify("已传送到木头人起点")
    end
end)

RedLightMainBox:AddButton("修复断腿", function()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end

    hum.PlatformStand = false
    hum:ChangeState(Enum.HumanoidStateType.GettingUp)
    hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, true)
    hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
    hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)

    for _, c in pairs(root:GetChildren()) do
        if c:IsA("BallSocketConstraint") then c:Destroy() end
    end
    for _, tagName in ipairs({"Ragdoll", "Stun", "RotateDisabled", "RagdollWakeupImmunity"}) do
        local tag = char:FindFirstChild(tagName)
        if tag then tag:Destroy() end
    end
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") and part:FindFirstChild("BoneCustom") then
            part.BoneCustom:Destroy()
        end
    end
    Notify("已修复断腿状态")
end)

RedLightUtilBox:AddToggle("GodMode", {
    Text = "上帝模式（抬升到安全高度）",
    Default = false,
    Callback = function(State)
        local rp = GetRoot()
        if not rp then return end
        local pos = rp.Position
        if State then
            rp.CFrame = CFrame.new(Vector3.new(pos.X, 1186.41, pos.Z))
        else
            rp.CFrame = CFrame.new(Vector3.new(pos.X, 1024.51, pos.Z))
        end
    end
})

RedLightUtilBox:AddToggle("HelpPlayerLoop", {
    Text = "帮助玩家（循环）",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Toggles.HelpPlayerLoop and Toggles.HelpPlayerLoop.Value do
                local char = LocalPlayer.Character
                if char then
                    local root = char:FindFirstChild("HumanoidRootPart")
                    if root then
                        for _, player in ipairs(Players:GetPlayers()) do
                            if player ~= LocalPlayer and player.Character then
                                local pHum = player.Character:FindFirstChildOfClass("Humanoid")
                                local pRoot = player.Character:FindFirstChild("HumanoidRootPart")
                                if pHum and pHum.Health > 0 and pRoot then
                                    if (pRoot.Position - root.Position).Magnitude < 8 then
                                        pcall(function()
                                            pRoot.CFrame = pRoot.CFrame + Vector3.new(0, 0.5, 0)
                                        end)
                                    end
                                end
                            end
                        end
                    end
                end
                task.wait(0.2)
            end
        end)
    end
})

RedLightUtilBox:AddToggle("AutoCollectBandage", {
    Text = "自动收集绷带",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Toggles.AutoCollectBandage and Toggles.AutoCollectBandage.Value do
                local effects = Workspace:FindFirstChild("Effects")
                local root = GetRoot()
                if effects and root then
                    for _, obj in ipairs(effects:GetChildren()) do
                        if obj:IsA("Model") and obj.Name:lower():find("bandage") then
                            local pivot = obj:GetPivot()
                            if (pivot.Position - root.Position).Magnitude < 50 then
                                root.CFrame = CFrame.new(pivot.Position + Vector3.new(0, 3, 0))
                                task.wait(0.1)
                                pcall(function()
                                    local prompt = obj:FindFirstChildOfClass("ProximityPrompt", true)
                                    if prompt then fireproximityprompt(prompt) end
                                end)
                            end
                        end
                    end
                end
                task.wait(0.3)
            end
        end)
    end
})

-- ==========================================================
-- 糖饼
-- ==========================================================
local DalgonaPlayerBox = DalgonaTab:AddLeftGroupbox("玩家功能")
local DalgonaUtilBox = DalgonaTab:AddRightGroupbox("玩家工具")

local function GetClickEvent()
    local rep = ReplicatedStorage:FindFirstChild("Replication")
    return rep and rep:FindFirstChild("Event")
end

local function GetDalgonaRemote()
    return FindRemote({"DALGONATEMPREMPTE", "DalgonaRemote", "DalgonaQTE", "DalgonaEvent"})
end

local function FixCamera()
    local char = LocalPlayer.Character
    if not char then return end
    local cam = Workspace.CurrentCamera
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.CameraOffset = Vector3.new(0, 0, 0)
        hum.AutoRotate = true
    end
    cam.CameraType = Enum.CameraType.Custom
    cam.CameraSubject = hum
    pcall(function()
        cam.CFrame = CFrame.new(char:FindFirstChild("HumanoidRootPart").Position + Vector3.new(0, 2, 5))
    end)
end

DalgonaPlayerBox:AddButton("自动扣糖饼", function()
    task.spawn(function()
        local values = Workspace:FindFirstChild("Values")
        if values then
            local cg = values:FindFirstChild("CurrentGame")
            if cg and cg.Value ~= "Dalgona" then
                Notify("只在糖饼游戏期间有效！")
                return
            end
        end
        local dr = GetDalgonaRemote()
        if not dr then Notify("未找到糖饼远程，请稍后重试"); return end
        local ev = GetClickEvent()
        for i = 1, 25 do
            if ev then pcall(function() ev:FireServer("Clicked") end) end
            pcall(function() dr:FireServer() end)
            pcall(function() dr:FireServer({ Clicked = true }) end)
            task.wait(0.08)
        end
        pcall(function() dr:FireServer({ Completed = true }) end)
        task.wait(3)
        pcall(FixCamera)
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                for i = 1, 5 do
                    hum:ChangeState(Enum.HumanoidStateType.Jumping)
                    task.wait(0.3)
                end
            end
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    pcall(function()
                        part.Color = Color3.fromRGB(0, 255, 0)
                        part.Material = Enum.Material.SmoothPlastic
                    end)
                end
            end
        end
        Notify("扣糖饼 + 修复相机完成 ✅")
    end)
end)

DalgonaPlayerBox:AddButton("修复相机", FixCamera)

DalgonaPlayerBox:AddButton("免费打火机", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/ergergq2/erge.github.io/refs/heads/main/free.lua"))()
end)

DalgonaPlayerBox:AddDivider()
DalgonaPlayerBox:AddLabel("注意：防推挤\n并非总是有效")

DalgonaUtilBox:AddButton("防推挤（测试版）", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/eruiier/antipush.github.io/refs/heads/main/ringta.lua"))()
end)

-- ==========================================================
-- 拔河（自动尝试多种远程+参数）
-- ==========================================================
local TugOfWarBox = TugOfWarTab:AddLeftGroupbox("拔河功能")
local TugOfWarUtilBox = TugOfWarTab:AddRightGroupbox("拔河工具")

local PerfectPull = false

local function FireTugOfWarRemote(perfect)
    local remotes = {
        ReplicatedStorage:FindFirstChild("Remotes"),
        ReplicatedStorage:FindFirstChild("RemoteEvents"),
        ReplicatedStorage,
    }
    local names = {
        "TemporaryReachedBindable",
        "TugOfWarRemote",
        "TugRemote",
        "TugOfWar",
        "QTERemote",
        "PullRemote",
        "QTE",
        "GameRemote",
        "MainRemote",
        "QTEEvent",
    }
    local argVariants = {
        { PerfectQTE = perfect },
        { perfect },
        { perfect, "QTE" },
        { "QTE", perfect },
        { Success = perfect },
        {},
    }
    local found = false
    for _, root in ipairs(remotes) do
        if root then
            for _, name in ipairs(names) do
                local remote = root:FindFirstChild(name)
                if remote and (remote:IsA("RemoteEvent") or remote:IsA("BindableEvent")) then
                    found = true
                    for _, args in ipairs(argVariants) do
                        pcall(function() remote:FireServer(args) end)
                    end
                end
            end
        end
    end
    return found
end

TugOfWarBox:AddToggle("AutoPull", {
    Text = "自动拉绳",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            local warned = false
            while Toggles.AutoPull and Toggles.AutoPull.Value do
                local ok = FireTugOfWarRemote(PerfectPull)
                if not ok and not warned then
                    Notify("未找到拔河远程事件，请用右侧“探测远程”查看名字", 5)
                    warned = true
                end
                task.wait(0.05)
            end
        end)
    end
})

TugOfWarBox:AddToggle("PerfectPull", {
    Text = "完美拉绳",
    Default = false,
    Callback = function(State)
        PerfectPull = State
    end
})

TugOfWarUtilBox:AddButton("探测远程事件（输出到控制台）", function()
    Notify("正在输出所有远程事件到控制台（F9）")
    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("BindableEvent") or obj:IsA("RemoteFunction") then
            print("[远程]", obj:GetFullName(), obj.ClassName)
        end
    end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("BindableEvent") then
            print("[远程-WS]", obj:GetFullName(), obj.ClassName)
        end
    end
end)

TugOfWarUtilBox:AddButton("传送到拔河场地中心", function()
    local rp = GetRoot()
    if rp then
        rp.CFrame = CFrame.new(Vector3.new(0, 500, 0))
        Notify("已抬升到安全高度，请根据需要调整")
    end
end)

-- ==========================================================
-- 捉迷藏
-- ==========================================================
local HideSeekFeaturesBox = HideAndSeekTab:AddLeftGroupbox("捉迷藏功能")
local HideSeekUtilBox = HideAndSeekTab:AddRightGroupbox("捉迷藏工具")
local HideSeekESPBox = HideAndSeekTab:AddLeftGroupbox("捉迷藏透视")

local SeekerColor = Color3.new(1, 0, 0)
local HiderColor  = Color3.new(0, 0.5, 1)

local function GetTeamInfo(player)
    if player:GetAttribute("IsHider") then return HiderColor, "躲藏者" end
    if player.TeamColor then
        local tc = player.TeamColor
        if tc == BrickColor.new("Bright blue") or tc == BrickColor.new("Cyan") or tc == BrickColor.new("Blue") then
            return HiderColor, "躲藏者"
        elseif tc == BrickColor.new("Bright red") or tc == BrickColor.new("Red") then
            return SeekerColor, "抓捕者"
        end
    end
    if player.Character then
        local torso = player.Character:FindFirstChild("Torso") or player.Character:FindFirstChild("UpperTorso")
        if torso and torso:IsA("BasePart") then
            if torso.Color.B > torso.Color.R then return HiderColor, "躲藏者"
            elseif torso.Color.R > torso.Color.B then return SeekerColor, "抓捕者" end
        end
    end
    return nil, nil
end

local ESPConnections = {}

local function ClearAllESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local esp = head:FindFirstChild("RoleESP")
                if esp then esp:Destroy() end
            end
            for _, part in ipairs(player.Character:GetChildren()) do
                if part:IsA("BasePart") then
                    local eb = part:FindFirstChild("ESPBox")
                    if eb then eb:Destroy() end
                end
            end
        end
    end
end

local function CreatePlayerESP(targetPlayer)
    if targetPlayer == LocalPlayer then return end
    if not targetPlayer.Character then return end
    local char = targetPlayer.Character
    local head = char:FindFirstChild("Head")
    if not head then return end

    local existing = head:FindFirstChild("RoleESP")
    if existing then existing:Destroy() end
    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") and part:FindFirstChild("ESPBox") then
            part.ESPBox:Destroy()
        end
    end

    local color, roleText = GetTeamInfo(targetPlayer)
    if not color then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "RoleESP"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 60, 0, 20)
    billboard.StudsOffset = Vector3.new(0, 1, 0)
    billboard.AlwaysOnTop = true

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundColor3 = color
    frame.BackgroundTransparency = 0.3
    frame.BorderSizePixel = 0
    frame.Parent = billboard

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = roleText
    label.TextColor3 = color
    label.TextStrokeTransparency = 0.5
    label.Font = Enum.Font.SourceSansBold
    label.TextScaled = true
    label.Parent = billboard
    billboard.Parent = head

    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESPBox"
            box.Size = part.Size
            box.Adornee = part
            box.AlwaysOnTop = true
            box.ZIndex = 10
            box.Transparency = 0.5
            box.Color3 = color
            box.Parent = part
        end
    end
end

HideSeekESPBox:AddToggle("EspHiderSeeker", {
    Text = "透视躲藏者 & 抓捕者",
    Default = false,
    Callback = function(State)
        for _, conn in ipairs(ESPConnections) do conn:Disconnect() end
        ESPConnections = {}
        if State then
            ClearAllESP()
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    CreatePlayerESP(player)
                end
            end
            local c1 = Players.PlayerAdded:Connect(function(player)
                if Toggles.EspHiderSeeker.Value then
                    player.CharacterAdded:Connect(function()
                        task.wait(0.1)
                        if Toggles.EspHiderSeeker.Value then CreatePlayerESP(player) end
                    end)
                end
            end)
            local c2 = Players.PlayerRemoving:Connect(function(player)
                if player.Character then
                    local head = player.Character:FindFirstChild("Head")
                    if head then
                        local esp = head:FindFirstChild("RoleESP")
                        if esp then esp:Destroy() end
                    end
                end
            end)
            table.insert(ESPConnections, c1)
            table.insert(ESPConnections, c2)
        else
            ClearAllESP()
        end
    end
})

local TrapRoomPos = Vector3.new(191, 989, 154)
local AutoKillConn = nil
local AutoKillTarget = nil

local function GetHiderTarget()
    local myRoot = GetRoot()
    if not myRoot then return nil end
    local closest, minDist = nil, math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            local hrp = player.Character:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 and hrp then
                local _, role = GetTeamInfo(player)
                local isHider = player:GetAttribute("IsHider") or role == "躲藏者"
                if isHider and (hrp.Position - TrapRoomPos).Magnitude >= 5 then
                    local d = (myRoot.Position - hrp.Position).Magnitude
                    if d < minDist then minDist = d; closest = player end
                end
            end
        end
    end
    return closest
end

local function AutoKillHidersStart()
    if AutoKillConn then AutoKillConn:Disconnect() end
    AutoKillTarget = GetHiderTarget()
    if not AutoKillTarget then Notify("未找到躲藏者"); return end
    AutoKillConn = RunService.Heartbeat:Connect(function()
        if not Toggles.AutoKillHiders or not Toggles.AutoKillHiders.Value then
            if AutoKillConn then AutoKillConn:Disconnect() end
            AutoKillConn = nil; return
        end
        if not AutoKillTarget or not AutoKillTarget.Character then
            AutoKillTarget = GetHiderTarget()
            if not AutoKillTarget then return end
        end
        local tHRP = AutoKillTarget.Character:FindFirstChild("HumanoidRootPart")
        local myHRP = GetRoot()
        if not tHRP or not myHRP then return end
        local topPos = tHRP.Position + Vector3.new(0, 7, 0)
        myHRP.CFrame = CFrame.new(topPos) * CFrame.Angles(math.rad(-90), 0, 0)
    end)
end

HideSeekFeaturesBox:AddToggle("AutoKillHiders", {
    Text = "自动杀死躲藏者",
    Default = false,
    Callback = function(State)
        if State then AutoKillHidersStart()
        else
            if AutoKillConn then AutoKillConn:Disconnect() end
            AutoKillConn = nil; AutoKillTarget = nil
        end
    end
})

HideSeekFeaturesBox:AddToggle("KillAuraSafe", {
    Text = "击杀光环（极安全）",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Toggles.KillAuraSafe and Toggles.KillAuraSafe.Value do
                local myRoot = GetRoot()
                if myRoot then
                    for _, player in ipairs(Players:GetPlayers()) do
                        if player ~= LocalPlayer and player.Character then
                            local pHum = player.Character:FindFirstChildOfClass("Humanoid")
                            local pRoot = player.Character:FindFirstChild("HumanoidRootPart")
                            if pHum and pHum.Health > 0 and pRoot then
                                local _, role = GetTeamInfo(player)
                                local isHider = player:GetAttribute("IsHider") or role == "躲藏者"
                                if isHider and (pRoot.Position - myRoot.Position).Magnitude < 10 then
                                    local knife = LocalPlayer.Backpack:FindFirstChild("Knife") or LocalPlayer.Character:FindFirstChild("Knife")
                                    if knife and not LocalPlayer.Character:FindFirstChild("Knife") then
                                        pcall(function() knife.Parent = LocalPlayer.Character end)
                                    end
                                end
                            end
                        end
                    end
                end
                task.wait(0.2)
            end
        end)
    end
})

HideSeekFeaturesBox:AddToggle("ExitDoorESP", {
    Text = "透视逃生门",
    Default = false,
    Callback = function(State)
        local function Create()
            local map = workspace:FindFirstChild("HideAndSeekMap")
            if not map then return end
            local fd = map:FindFirstChild("NEWFIXEDDOORS")
            if not fd then return end
            for _, floor in ipairs(fd:GetChildren()) do
                for _, door in ipairs(floor:GetDescendants()) do
                    if door.Name:lower():find("exitdoor") then
                        local bp = door:FindFirstChildOfClass("BasePart") or door
                        if bp:IsA("BasePart") and not bp:FindFirstChild("ExitDoorESP") then
                            local h = Instance.new("Highlight")
                            h.Name = "ExitDoorESP"
                            h.Parent = bp
                            h.FillColor = Color3.fromRGB(0, 128, 255)
                            h.OutlineColor = Color3.fromRGB(0, 128, 255)
                            h.FillTransparency = 0.3
                            h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                            local bb = Instance.new("BillboardGui")
                            bb.Adornee = bp
                            bb.Size = UDim2.new(0, 100, 0, 25)
                            bb.StudsOffset = Vector3.new(0, 2, 0)
                            bb.AlwaysOnTop = true
                            bb.Parent = bp
                            local lb = Instance.new("TextLabel")
                            lb.BackgroundTransparency = 1
                            lb.Size = UDim2.new(1, 0, 1, 0)
                            lb.Text = "出口门"
                            lb.TextColor3 = Color3.fromRGB(0, 128, 255)
                            lb.TextScaled = true
                            lb.Font = Enum.Font.SourceSansBold
                            lb.Parent = bb
                        end
                    end
                end
            end
        end
        if State then Create()
        else
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj.Name == "ExitDoorESP" then obj:Destroy() end
            end
        end
    end
})

HideSeekFeaturesBox:AddToggle("ESPKeys", {
    Text = "透视钥匙",
    Default = false,
    Callback = function(State)
        if not State then return end
        local effects = Workspace:FindFirstChild("Effects")
        if not effects then return end
        for _, obj in pairs(effects:GetChildren()) do
            if obj:IsA("Model") and obj.PrimaryPart and obj.Name:find("DroppedKey") then
                local h = Instance.new("Highlight")
                h.Name = "KeyESP"
                h.Adornee = obj
                h.FillColor = Color3.fromRGB(255, 255, 0)
                h.OutlineColor = Color3.fromRGB(255, 215, 0)
                h.FillTransparency = 0.3
                h.Parent = obj
            end
        end
    end
})

HideSeekUtilBox:AddButton("向上传送 100 格", function()
    local rp = GetRoot()
    if rp then rp.CFrame = CFrame.new(rp.Position + Vector3.new(0, 100, 0)) end
end)

HideSeekUtilBox:AddButton("向下传送 40 格", function()
    local rp = GetRoot()
    if rp then rp.CFrame = CFrame.new(rp.Position - Vector3.new(0, 40, 0)) end
end)

HideSeekUtilBox:AddButton("传送到陷阱房", function()
    local rp = GetRoot()
    if rp then rp.CFrame = CFrame.new(TrapRoomPos); Notify("已传送到陷阱房") end
end)

HideSeekUtilBox:AddButton("生成尖刺防坠落平台", function()
    CreatePlatform("InkHub_TrapPlatform", Vector3.new(191, 999, 154), Vector3.new(50, 1, 50))
    Notify("已生成防坠落平台")
end)

HideSeekUtilBox:AddButton("删除尖刺", function()
    local map = Workspace:FindFirstChild("HideAndSeekMap")
    if map then
        local kp = map:FindFirstChild("KillingParts")
        if kp then kp:Destroy(); Notify("已删除尖刺") end
    end
end)

HideSeekUtilBox:AddButton("传送到随机躲藏者", function()
    local target = GetHiderTarget()
    if target and target.Character then
        local root = target.Character:FindFirstChild("HumanoidRootPart")
        local myRoot = GetRoot()
        if root and myRoot then myRoot.CFrame = root.CFrame + Vector3.new(0, 2, 0) end
    end
end)

-- ==========================================================
-- 跳绳 / 玻璃桥
-- ==========================================================
local JumpRopeBox = JumpRopeTab:AddLeftGroupbox("跳绳")
local GlassBridgeBox = JumpRopeTab:AddRightGroupbox("玻璃桥")

JumpRopeBox:AddButton("传送到终点（跳绳）", function()
    LocalPlayer.Character:PivotTo(CFrame.new(Vector3.new(720.83, 202.7, 921.26)))
end)

JumpRopeBox:AddButton("删除绳子 + 防坠落平台", function()
    local effects = Workspace:FindFirstChild("Effects")
    if effects and effects:FindFirstChild("rope") then effects.rope:Destroy() end
    CreatePlatform("InkHub_RopePlatform", Vector3.new(672.41, 190.24, 920.59), Vector3.new(100, 1, 100))
end)

local GlassESP = { Active = false, OriginalColors = {} }
local FixedGlassPlatform = nil

local function UpdateGlassColors()
结束
如果 redPos（you mayoto FixedGlassPlatform. Parent）
FixedGlassPlatform = CreatePlatform("FixedGlassAntiFallPlatform", Vector3.new(redPos.X, redPos.Y + 0.5, redPos.Z), Vector3.new(1000, 1, 1000), Color3.fromRGB(255, 255, 255))
结束
结束
局部函数 RestoreGlassColors（）
对于部分，我喜欢（GlassES.OriginalColor）
如果分开
pcall(function() part.Color = color; part.Transparency = 0 end)
结束
结束
GlassESP.OriginalColors = {}
如果 FixedGlassPlatform，you worto FixedGlassPlatform:Destroy（）；FixedGlassPlatform=nil end[结束]GlassBridgeBox:AddButton（“传送到终点（传送到终点）”
如果不是GlassES.OriginalColors[part][字符：PivotTo（CFrame. new）Vector3. new（-203.9，520.7，-1534）] = part.Color
GlassESP.OriginalColors[part] = part.Color
结束
如果 redPos you mayoto FixedGlassPlatform
FixedGlassPlatform = CreatePlatform("FixedGlassAntiFallPlatform", Vector3.new(redPos.X, redPos.Y + 0.5, redPos.Z), Vector3.new(1000, 1, 1000), Color3.fromRGB(255, 255, 255))
结束
结束
局部函数 RestoreGlassColors（）
对于部分，我想知道（GlassES.OriginalColor）
结束
= part.Color
FixedGlassPlatform = CreatePlatform("FixedGlassAntiFallPlatform", Vector3.new(redPos.X, redPos.Y + 0.5, redPos.Z), Vector3.new(1000, 1, 1000), Color3.fromRGB(255, 255, 255))
结束

结束
局部函数 RestoreGlassColors（）
对于部分，我喜欢（GlassES.OriginalColor）
结束
结束
结束透明度=0.3
本地 pp=tm
本地 isB=pp:GetAttribute（“exploitingisevil”）=======true
local Color=isB和 Color3.fromRGB(255,000000,0)Color3.fromRGB(0，255,0)

[如果是 isB you mayor redPos，redPos=pp you]
对于_，part in pay(tm:getsolutions（）)
