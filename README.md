-- language: lua, file: cape_control_panel_smooth.lua, target: Roblox LocalScript / Executor
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local ContentProvider = game:GetService("ContentProvider")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local isAlive = false
local currentEvent = nil

local isMainOn = false
local mode = "Color"
local colorSubMode = "All"
local delayTime = 0.016

local imageCount = 1
local imageInputs = {}
local selectedColors = {}
local colorListItems = {}

if PlayerGui:FindFirstChild("AXR_CapeController") then
    PlayerGui.AXR_CapeController:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AXR_CapeController"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 260, 0, 160)
mainFrame.Position = UDim2.new(0.05, 0, 0.25, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 33)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

local function createToggle(name, text, positionY, initialText, initialColor)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(0.9, 0, 0, 32)
    btn.Position = UDim2.new(0.05, 0, 0, positionY)
    btn.BackgroundColor3 = initialColor
    btn.Text = text .. ": " .. initialText
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamSemibold
    btn.AutoButtonColor = false
    btn.Parent = mainFrame

    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    return btn
end

local mainToggle = createToggle("MainToggle", "반복 변경", 15, "OFF", Color3.fromRGB(180, 50, 50))
local modeToggle = createToggle("ModeToggle", "모드 선택", 53, "색상 바꾸기", Color3.fromRGB(50, 100, 180))

local colorSubToggle = Instance.new("TextButton")
colorSubToggle.Name = "ColorSubToggle"
colorSubToggle.Size = UDim2.new(0.9, 0, 0, 28)
colorSubToggle.Position = UDim2.new(0.05, 0, 0, 91)
colorSubToggle.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
colorSubToggle.Text = "색상 방식: 전체 순환 (무지개)"
colorSubToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
colorSubToggle.TextSize = 11
colorSubToggle.Font = Enum.Font.GothamSemibold
colorSubToggle.AutoButtonColor = false
colorSubToggle.Visible = true
colorSubToggle.Parent = mainFrame
Instance.new("UICorner", colorSubToggle).CornerRadius = UDim.new(0, 6)

local sliderTitle = Instance.new("TextLabel")
sliderTitle.Size = UDim2.new(0.9, 0, 0, 18)
sliderTitle.Position = UDim2.new(0.05, 0, 0, 125)
sliderTitle.BackgroundTransparency = 1
sliderTitle.Text = "변경 속도: 초고속"
sliderTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
sliderTitle.TextSize = 11
sliderTitle.Font = Enum.Font.Gotham
sliderTitle.Parent = mainFrame

local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(0.9, 0, 0, 8)
sliderBg.Position = UDim2.new(0.05, 0, 0, 145)
sliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
sliderBg.BorderSizePixel = 0
sliderBg.Parent = mainFrame

Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(0, 4)

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(1, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderBg

Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(0, 4)

local sliderBtn = Instance.new("TextButton")
sliderBtn.Size = UDim2.new(1, 0, 1, 0)
sliderBtn.BackgroundTransparency = 1
sliderBtn.Text = ""
sliderBtn.Parent = sliderBg

-- 색상 선택 팔레트 및 동적 목록
local colorSection = Instance.new("Frame")
colorSection.Name = "ColorSection"
colorSection.Size = UDim2.new(0.9, 0, 0, 200)
colorSection.Position = UDim2.new(0.05, 0, 0, 125)
colorSection.BackgroundTransparency = 1
colorSection.Visible = false
colorSection.Parent = mainFrame

local paletteContainer = Instance.new("Frame")
paletteContainer.Size = UDim2.new(1, 0, 0, 60)
paletteContainer.BackgroundTransparency = 1
paletteContainer.Parent = colorSection

local colorPalette = {
    Color3.fromRGB(255, 50, 50),   -- 빨강
    Color3.fromRGB(255, 150, 50),  -- 주황
    Color3.fromRGB(255, 220, 50),  -- 노랑
    Color3.fromRGB(50, 220, 80),   -- 초록
    Color3.fromRGB(50, 150, 255),  -- 파랑
    Color3.fromRGB(150, 50, 255),  -- 보라
    Color3.fromRGB(255, 100, 200), -- 핑크
    Color3.fromRGB(255, 255, 255), -- 흰색
    Color3.fromRGB(30, 30, 30)     -- 검정
}

local colorGridLayout = Instance.new("UIGridLayout")
colorGridLayout.CellSize = UDim2.new(0, 24, 0, 24)
colorGridLayout.CellPadding = UDim2.new(0, 4, 0, 4)
colorGridLayout.Parent = paletteContainer

local selectedScroll = Instance.new("ScrollingFrame")
selectedScroll.Size = UDim2.new(1, 0, 0, 130)
selectedScroll.Position = UDim2.new(0, 0, 0, 65)
selectedScroll.BackgroundTransparency = 1
selectedScroll.BorderSizePixel = 0
selectedScroll.ScrollBarThickness = 4
selectedScroll.Parent = colorSection

local selectedListLayout = Instance.new("UIListLayout")
selectedListLayout.SortOrder = Enum.SortOrder.LayoutOrder
selectedListLayout.Padding = UDim.new(0, 5)
selectedListLayout.Parent = selectedScroll

local updateSelectedColorList

local function removeSelectedColor(index)
    local item = selectedColors[index]
    if item then
        if item.stroke then item.stroke.Enabled = false end
        table.remove(selectedColors, index)
        updateSelectedColorList()
    end
end

updateSelectedColorList = function()
    for _, elem in ipairs(colorListItems) do
        elem:Destroy()
    end
    table.clear(colorListItems)

    for i, item in ipairs(selectedColors) do
        local row = Instance.new("Frame")
        row.Size = UDim2.new(0.95, 0, 0, 28)
        row.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
        row.Parent = selectedScroll
        Instance.new("UICorner", row).CornerRadius = UDim.new(0, 4)

        local colorPreview = Instance.new("Frame")
        colorPreview.Size = UDim2.new(0, 18, 0, 18)
        colorPreview.Position = UDim2.new(0, 6, 0, 5)
        colorPreview.BackgroundColor3 = item.color
        colorPreview.Parent = row
        Instance.new("UICorner", colorPreview).CornerRadius = UDim.new(0, 3)

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.6, 0, 1, 0)
        label.Position = UDim2.new(0, 30, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = i .. "번 색상"
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.TextSize = 11
        label.Font = Enum.Font.Gotham
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = row

        local delBtn = Instance.new("TextButton")
        delBtn.Size = UDim2.new(0, 22, 0, 22)
        delBtn.Position = UDim2.new(1, -26, 0, 3)
        delBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
        delBtn.Text = "X"
        delBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        delBtn.TextSize = 11
        delBtn.Font = Enum.Font.GothamBold
        delBtn.Parent = row
        Instance.new("UICorner", delBtn).CornerRadius = UDim.new(0, 4)

        delBtn.MouseButton1Click:Connect(function()
            removeSelectedColor(i)
        end)

        table.insert(colorListItems, row)
    end

    selectedScroll.CanvasSize = UDim2.new(0, 0, 0, #selectedColors * 33)

    if mode == "Color" and colorSubMode == "Custom" then
        local targetHeight = math.min(130, math.max(60, #selectedColors * 33))
        selectedScroll.Size = UDim2.new(1, 0, 0, targetHeight)
        sliderTitle.Position = UDim2.new(0.05, 0, 0, 195 + targetHeight)
        sliderBg.Position = UDim2.new(0.05, 0, 0, 215 + targetHeight)
        mainFrame.Size = UDim2.new(0, 260, 0, 235 + targetHeight)
    end
end

for _, c in ipairs(colorPalette) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 24, 0, 24)
    btn.BackgroundColor3 = c
    btn.Text = ""
    btn.AutoButtonColor = false
    btn.Parent = paletteContainer

    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(255, 255, 255)
    stroke.Enabled = false
    stroke.Parent = btn

    btn.MouseButton1Click:Connect(function()
        local foundIdx = nil
        for idx, item in ipairs(selectedColors) do
            if item.color == c then
                foundIdx = idx
                break
            end
        end

        if foundIdx then
            removeSelectedColor(foundIdx)
        else
            table.insert(selectedColors, {color = c, stroke = stroke})
            stroke.Enabled = true
            updateSelectedColorList()
        end
    end)
end

-- 이미지 선택 섹션
local imageSection = Instance.new("Frame")
imageSection.Name = "ImageSection"
imageSection.Size = UDim2.new(0.9, 0, 0, 0)
imageSection.Position = UDim2.new(0.05, 0, 0, 125)
imageSection.BackgroundTransparency = 1
imageSection.Visible = false
imageSection.Parent = mainFrame

local countFrame = Instance.new("Frame")
countFrame.Size = UDim2.new(1, 0, 0, 28)
countFrame.BackgroundTransparency = 1
countFrame.Parent = imageSection

local countLabel = Instance.new("TextLabel")
countLabel.Size = UDim2.new(0.5, 0, 1, 0)
countLabel.BackgroundTransparency = 1
countLabel.Text = "개수: 1개"
countLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
countLabel.TextSize = 11
countLabel.Font = Enum.Font.Gotham
countLabel.TextXAlignment = Enum.TextXAlignment.Left
countLabel.Parent = countFrame

local minusBtn = Instance.new("TextButton")
minusBtn.Size = UDim2.new(0, 26, 0, 24)
minusBtn.Position = UDim2.new(0.65, 0, 0, 2)
minusBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
minusBtn.Text = "-"
minusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minusBtn.TextSize = 14
minusBtn.Font = Enum.Font.GothamBold
minusBtn.Parent = countFrame

local plusBtn = Instance.new("TextButton")
plusBtn.Size = UDim2.new(0, 26, 0, 24)
plusBtn.Position = UDim2.new(0.85, 0, 0, 2)
plusBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
plusBtn.Text = "+"
plusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
plusBtn.TextSize = 14
plusBtn.Font = Enum.Font.GothamBold
plusBtn.Parent = countFrame

Instance.new("UICorner", minusBtn).CornerRadius = UDim.new(0, 4)
Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 4)

local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, 0, 0, 120)
scrollFrame.Position = UDim2.new(0, 0, 0, 32)
scrollFrame.BackgroundTransparency = 1
scrollFrame.BorderSizePixel = 0
scrollFrame.ScrollBarThickness = 4
scrollFrame.Parent = imageSection

local uiListLayout = Instance.new("UIListLayout")
uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayout.Padding = UDim.new(0, 5)
uiListLayout.Parent = scrollFrame

local function updateImageInputs()
    for _, box in ipairs(imageInputs) do
        box:Destroy()
    end
    table.clear(imageInputs)

    for i = 1, imageCount do
        local box = Instance.new("TextBox")
        box.Name = "ImageInput_" .. i
        box.Size = UDim2.new(0.95, 0, 0, 26)
        box.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
        box.PlaceholderText = "이미지 ID #" .. i .. " 입력"
        box.PlaceholderColor3 = Color3.fromRGB(120, 120, 130)
        box.Text = ""
        box.TextColor3 = Color3.fromRGB(255, 255, 255)
        box.TextSize = 11
        box.Font = Enum.Font.Code
        box.ClearTextOnFocus = false
        box.Parent = scrollFrame

        Instance.new("UICorner", box).CornerRadius = UDim.new(0, 4)

        box.FocusLost:Connect(function()
            local idStr = box.Text
            if idStr ~= "" then
                local formatted = tonumber(idStr) and ("rbxassetid://" .. idStr) or idStr
                task.spawn(function()
                    pcall(function()
                        ContentProvider:PreloadAsync({formatted})
                    end)
                end)
            end
        end)

        table.insert(imageInputs, box)
    end

    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, imageCount * 31)
    countLabel.Text = "이미지 개수: " .. imageCount .. "개"

    if mode == "Image" then
        local targetHeight = math.min(120, imageCount * 31)
        scrollFrame.Size = UDim2.new(1, 0, 0, targetHeight)
        mainFrame.Size = UDim2.new(0, 260, 0, 135 + targetHeight)
    end
end

mainToggle.MouseButton1Click:Connect(function()
    isMainOn = not isMainOn
    if isMainOn then
        mainToggle.Text = "반복 변경: ON"
        TweenService:Create(mainToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 180, 80)}):Play()
    else
        mainToggle.Text = "반복 변경: OFF"
        TweenService:Create(mainToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(180, 50, 50)}):Play()
    end
end)

modeToggle.MouseButton1Click:Connect(function()
    if mode == "Color" then
        mode = "Image"
        modeToggle.Text = "모드 선택: 이미지 바꾸기"
        TweenService:Create(modeToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(140, 60, 180)}):Play()
        colorSubToggle.Visible = false
        colorSection.Visible = false
        sliderTitle.Position = UDim2.new(0.05, 0, 0, 91)
        sliderBg.Position = UDim2.new(0.05, 0, 0, 111)
        imageSection.Visible = true
        imageCount = 1
        updateImageInputs()
    else
        mode = "Color"
        modeToggle.Text = "모드 선택: 색상 바꾸기"
        TweenService:Create(modeToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 100, 180)}):Play()
        colorSubToggle.Visible = true
        imageSection.Visible = false
        if colorSubMode == "All" then
            colorSection.Visible = false
            sliderTitle.Position = UDim2.new(0.05, 0, 0, 125)
            sliderBg.Position = UDim2.new(0.05, 0, 0, 145)
            mainFrame.Size = UDim2.new(0, 260, 0, 160)
        else
            colorSection.Visible = true
            updateSelectedColorList()
        end
    end
end)

colorSubToggle.MouseButton1Click:Connect(function()
    if colorSubMode == "All" then
        colorSubMode = "Custom"
        colorSubToggle.Text = "색상 방식: 특정 색 지정"
        TweenService:Create(colorSubToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(120, 70, 140)}):Play()
        colorSection.Visible = true
        updateSelectedColorList()
    else
        colorSubMode = "All"
        colorSubToggle.Text = "색상 방식: 전체 순환 (무지개)"
        TweenService:Create(colorSubToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(70, 70, 90)}):Play()
        colorSection.Visible = false
        sliderTitle.Position = UDim2.new(0.05, 0, 0, 125)
        sliderBg.Position = UDim2.new(0.05, 0, 0, 145)
        mainFrame.Size = UDim2.new(0, 260, 0, 160)
    end
end)

minusBtn.MouseButton1Click:Connect(function()
    if imageCount > 1 then
        imageCount = imageCount - 1
        if mode == "Image" then
            updateImageInputs()
        end
    end
end)

plusBtn.MouseButton1Click:Connect(function()
    if imageCount < 10 then
        imageCount = imageCount + 1
        if mode == "Image" then
            updateImageInputs()
        end
    end
end)

local dragging = false
sliderBtn.MouseButton1Down:Connect(function() dragging = true end)
game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local mousePos = input.Position.X
        local bgPos = sliderBg.AbsolutePosition.X
        local bgWidth = sliderBg.AbsoluteSize.X
        local percent = math.clamp((mousePos - bgPos) / bgWidth, 0, 1)
        
        delayTime = math.clamp(0.15 - (percent * 0.145), 0.005, 0.15)
        sliderFill.Size = UDim2.new(percent, 0, 1, 0)
        sliderTitle.Text = string.format("변경 속도: %.3fs", delayTime)
    end
end)

local function setupCharacter(char)
    isAlive = false
    currentEvent = nil

    local humanoid = char:WaitForChild("Humanoid", 10)
    if humanoid then
        humanoid.Died:Connect(function()
            isAlive = false
            currentEvent = nil
        end)
    end

    local event = char:WaitForChild("Communicate", 10)
    if event then
        task.wait(1)
        currentEvent = event
        isAlive = true
    end
end

if LocalPlayer.Character then task.spawn(setupCharacter, LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(function(newChar) task.spawn(setupCharacter, newChar) end)

local lastUpdate = 0
local hue = 0
local index = 1

RunService.RenderStepped:Connect(function(deltaTime)
    if not (isMainOn and isAlive and currentEvent and currentEvent.Parent) then return end

    lastUpdate = lastUpdate + deltaTime
    if lastUpdate >= delayTime then
        lastUpdate = 0

        if mode == "Color" then
            if colorSubMode == "All" then
                -- 레인보우 그라데이션 연출
                hue = (hue + 0.02) % 1
                local rainbowColor = Color3.fromHSV(hue, 1, 1)

                pcall(function()
                    currentEvent:FireServer({
                        OldData = "[]",
                        Goal = "Update Cape",
                        Color = rainbowColor,
                        Value = 1,
                        Decal = ""
                    })
                end)
            else
                -- 특정 지정된 색상 순환 (최소 2개 이상 선택되어 있을 때 작동)
                if #selectedColors >= 2 then
                    index = (index % #selectedColors) + 1
                    local targetColor = selectedColors[index].color

                    pcall(function()
                        currentEvent:FireServer({
                            OldData = "[]",
                            Goal = "Update Cape",
                            Color = targetColor,
                            Value = 1,
                            Decal = ""
                        })
                    end)
                elseif #selectedColors == 1 then
                    local targetColor = selectedColors[1].color
                    pcall(function()
                        currentEvent:FireServer({
                            OldData = "[]",
                            Goal = "Update Cape",
                            Color = targetColor,
                            Value = 1,
                            Decal = ""
                        })
                    end)
                end
            end

        elseif mode == "Image" then
            if #imageInputs > 0 then
                index = (index % #imageInputs) + 1
                local targetBox = imageInputs[index]
                local rawText = targetBox and targetBox.Text or ""
                
                local decalValue = ""
                if rawText ~= "" then
                    decalValue = tonumber(rawText) and ("rbxassetid://" .. rawText) or rawText
                end

                pcall(function()
                    currentEvent:FireServer({
                        OldData = "[]",
                        Goal = "Update Cape",
                        Color = Color3.new(1, 1, 1),
                        Value = 1,
                        Decal = decalValue
                    })
                end)
            end
        end
    end
end)
