-- classichub_track.lua
-- Hospede no GitHub e carregue com: loadstring(game:HttpGet("URL_RAW_AQUI"))()
-- Mostra contador de tempo no canto da tela + envia tracking ao painel
-- ==========================================================================

local HUB_TRACK_ENDPOINT = "https://leaderboard-manager--guzinsamuel10.replit.app/"

local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Player       = Players.LocalPlayer
local PlayerGui    = Player:WaitForChild("PlayerGui")

-- ===== HWID =====
local function getHwid()
    local ok, id = pcall(function()
        return tostring(game:GetService("RbxAnalyticsService"):GetClientId())
    end)
    return ok and id or "unknown"
end

-- ===== JSON =====
local function jsencode(t)
    local parts = {}
    for k, v in pairs(t) do
        local val
        if type(v) == "number" then val = tostring(v)
        elseif type(v) == "boolean" then val = tostring(v)
        else val = '"' .. tostring(v):gsub('"', '\\"') .. '"' end
        table.insert(parts, '"' .. tostring(k) .. '":' .. val)
    end
    return "{" .. table.concat(parts, ",") .. "}"
end

-- ===== HTTP =====
local function req(body)
    pcall(function()
        local http = request or (syn and syn.request) or (http and http.request)
        if not http then return end
        http({
            Url     = HUB_TRACK_ENDPOINT,
            Method  = "POST",
            Body    = jsencode(body),
            Headers = {["Content-Type"] = "application/json"}
        })
    end)
end

local function track(event, extra)
    extra = extra or {}
    local payload = {
        event    = event,
        username = Player.Name,
        userid   = Player.UserId,
        hwid     = getHwid(),
        game     = "Classic Hub",
        placeId  = game.PlaceId,
        ts       = os.time(),
    }
    for k, v in pairs(extra) do payload[k] = v end
    task.spawn(function() req(payload) end)
end

-- ===== CONTADOR VISUAL =====
local function formatTime(s)
    local h   = math.floor(s / 3600)
    local m   = math.floor((s % 3600) / 60)
    local sec = s % 60
    if h > 0 then
        return string.format("%d:%02d:%02d", h, m, sec)
    else
        return string.format("%02d:%02d", m, sec)
    end
end

local function createCounter()
    -- Remove instância anterior se existir
    local old = PlayerGui:FindFirstChild("CHubTracker")
    if old then old:Destroy() end

    local sg = Instance.new("ScreenGui")
    sg.Name          = "CHubTracker"
    sg.ResetOnSpawn  = false
    sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    sg.Parent        = PlayerGui

    local frame = Instance.new("Frame")
    frame.Size                = UDim2.new(0, 130, 0, 40)
    frame.Position            = UDim2.new(1, 20, 0, 12)  -- começa fora
    frame.BackgroundColor3    = Color3.fromRGB(10, 10, 15)
    frame.BackgroundTransparency = 0.2
    frame.BorderSizePixel     = 0
    frame.Parent              = sg

    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

    local stroke = Instance.new("UIStroke", frame)
    stroke.Color       = Color3.fromRGB(200, 0, 255)
    stroke.Thickness   = 1.5
    stroke.Transparency = 0.2

    -- Ícone
    local icon = Instance.new("TextLabel", frame)
    icon.Size                 = UDim2.new(0, 28, 1, 0)
    icon.Position             = UDim2.new(0, 6, 0, 0)
    icon.BackgroundTransparency = 1
    icon.Text                 = "⏱"
    icon.TextSize             = 15
    icon.Font                 = Enum.Font.GothamBold
    icon.TextColor3           = Color3.fromRGB(200, 0, 255)

    -- Tempo
    local label = Instance.new("TextLabel", frame)
    label.Size                = UDim2.new(1, -38, 1, 0)
    label.Position            = UDim2.new(0, 34, 0, 0)
    label.BackgroundTransparency = 1
    label.Text                = "00:00"
    label.TextSize            = 14
    label.Font                = Enum.Font.GothamBold
    label.TextColor3          = Color3.fromRGB(240, 240, 255)
    label.TextXAlignment      = Enum.TextXAlignment.Left

    -- Slide de entrada
    TweenService:Create(frame, TweenInfo.new(0.5, Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
        {Position = UDim2.new(1, -145, 0, 12)}):Play()

    return label
end

-- ===== INICIAR =====
local startedAt = os.time()
local label      = createCounter()

-- Atualiza contador todo segundo
task.spawn(function()
    while Player.Parent do
        task.wait(1)
        label.Text = formatTime(os.time() - startedAt)
    end
end)

-- Tracking
track("login")

task.spawn(function()
    while Player.Parent do
        task.wait(60)
        track("heartbeat", { duration = os.time() - startedAt })
    end
end)

game:BindToClose(function()
    track("stop", { duration = os.time() - startedAt })
end)
