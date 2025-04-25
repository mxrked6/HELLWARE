local cloneref = cloneref or function(ref)
    return ref
end

local GetService = game.GetService
local Services = setmetatable({}, {
    __index = function(self, Property)
        local success, service = pcall(GetService, game, Property)
        if success then
            self[Property] = cloneref(service)
            return service
        end
    end
})

local GetPlayers = Services.Players.GetPlayers
local JSONEncode, JSONDecode, GenerateGUID = 
    Services.HttpService.JSONEncode, 
    Services.HttpService.JSONDecode,
    Services.HttpService.GenerateGUID

local GetPropertyChangedSignal = game.GetPropertyChangedSignal
local GetChildren, GetDescendants = game.GetChildren, game.GetDescendants
local IsA = game.IsA
local FindFirstChild = game.FindFirstChild

local Tfind, sort, concat, pack, unpack = 
    table.find, 
    table.sort,
    table.concat,
    table.pack,
    table.unpack

local lower, split, sub, format = 
    string.lower,
    string.split, 
    string.sub,
    string.format

local random, floor, clamp = 
    math.random,
    math.floor,
    math.clamp

local Instancenew = Instance.new
local Vector2new = Vector2.new
local UDim2new = UDim2.new
local UDimnew = UDim.new
local Color3new = Color3.new
local Color3fromRGB = Color3.fromRGB
local Color3fromHSV = Color3.fromHSV
local ToHSV = Color3new().ToHSV

local Camera = Services.Workspace.CurrentCamera
local LocalPlayer = Services.Players.LocalPlayer
local Mouse = LocalPlayer and LocalPlayer:GetMouse()
local Destroy, Clone = game.Destroy, game.Clone
local Connection = game.Loaded
local CWait = Connection.Wait
local CConnect = Connection.Connect

local Disconnect
do
    local CalledConnection = CConnect(Connection, function() end)
    Disconnect = CalledConnection.Disconnect
end

local Connections = {}
local AddConnection = function(...)
    local ConnectionsToAdd = {...}
    for i = 1, #ConnectionsToAdd do
        Connections[#Connections + 1] = ConnectionsToAdd[i]
    end
    return ...
end

local UIElements = Services.InsertService:LoadLocalAsset("rbxassetid://6945229203")
local GuiObjects = UIElements.GuiObjects

local Colors = {
    Primary = Color3fromRGB(255, 85, 85),
    Secondary = Color3fromRGB(200, 60, 60),
    Background = Color3fromRGB(20, 20, 20),
    Element = Color3fromRGB(35, 35, 35),
    Border = Color3fromRGB(50, 50, 50),
    Text = Color3fromRGB(220, 220, 220),
    TextHover = Color3fromRGB(255, 255, 255),
    TextPressed = Color3fromRGB(180, 180, 180),
    Accent = Color3fromRGB(255, 100, 100),
    Shadow = Color3fromRGB(0, 0, 0)
}

local Theme = {
    AnimationSpeed = 0.2,
    ScrollSmoothing = 0.12,
    CornerRadius = UDimnew(0, 8),
    ShadowOpacity = 0.3,
    ShadowOffset = Vector2new(2, 2)
}

local Debounce = function(Func)
    local Debounce_ = false
    return function(...)
        if not Debounce_ then
            Debounce_ = true
            Func(...)
            Debounce_ = false
        end
    end
end

local Utils = {}

Utils.SmoothScroll = function(content, SmoothingFactor)
    content.ScrollingEnabled = false
    local input = Clone(content)
    
    input:ClearAllChildren()
    input.BackgroundTransparency = 1
    input.ScrollBarImageTransparency = 1
    input.ZIndex = content.ZIndex + 1
    input.Name = "_smoothinputframe"
    input.ScrollingEnabled = true
    input.Parent = content.Parent

    local function syncProperty(prop)
        AddConnection(CConnect(GetPropertyChangedSignal(content, prop), function()
            input[prop] = prop == "ZIndex" and content[prop] + 1 or content[prop]
        end))
    end

    for _, prop in ipairs({"CanvasSize", "Position", "Rotation", "ScrollingDirection", "ScrollBarThickness", 
                           "BorderSizePixel", "ElasticBehavior", "SizeConstraint", "ZIndex", 
                           "BorderColor3", "Size", "AnchorPoint", "Visible"}) do
        syncProperty(prop)
    end

    local smoothConnection = AddConnection(CConnect(Services.RunService.RenderStepped, function()
        local a = content.CanvasPosition
        local b = input.CanvasPosition
        local c = SmoothingFactor
        local d = (b - a) * c + a
        content.CanvasPosition = d
    end))

    AddConnection(CConnect(content.AncestryChanged, function()
        if not content.Parent then
            Destroy(input)
            Disconnect(smoothConnection)
        end
    end))
end

Utils.Tween = function(Object, Style, Direction, Time, Goal)
    local TInfo = TweenInfo.new(Time or Theme.AnimationSpeed, Enum.EasingStyle[Style], Enum.EasingDirection[Direction])
    local Tween = Services.TweenService:Create(Object, TInfo, Goal)
    Tween:Play()
    return Tween
end

Utils.MultColor3 = function(Color, Delta)
    return Color3new(clamp(Color.R * Delta, 0, 1), clamp(Color.G * Delta, 0, 1), clamp(Color.B * Delta, 0, 1))
end

Utils.AddShadow = function(Object)
    local Shadow = Instancenew("ImageLabel")
    Shadow.Name = "Shadow"
    Shadow.BackgroundTransparency = 1
    Shadow.Image = "rbxassetid://1316045217"
    Shadow.ImageColor3 = Colors.Shadow
    Shadow.ImageTransparency = Theme.ShadowOpacity
    Shadow.Size = UDim2new(1, 10, 1, 10)
    Shadow.Position = UDim2new(0, Theme.ShadowOffset.X, 0, Theme.ShadowOffset.Y)
    Shadow.ZIndex = Object.ZIndex - 1
    Shadow.Parent = Object
    return Shadow
end

Utils.Draggable = function(UI, DragUi)
    local DragToggle, DragStart, DragInput
    local StartPos
    DragUi = DragUi or UI

    local function UpdateInput(Input)
        local Delta = Input.Position - DragStart
        local Position = UDim2new(StartPos.X.Scale, StartPos.X.Offset + Delta.X, 
                                 StartPos.Y.Scale, StartPos.Y.Offset + Delta.Y)
        Utils.Tween(UI, "Sine", "Out", .15, {Position = Position})
    end

    AddConnection(CConnect(UI.InputBegan, function(Input)
        if (Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch) and 
           not Services.UserInputService:GetFocusedTextBox() then
            DragToggle = true
            DragStart = Input.Position
            StartPos = UI.Position

            AddConnection(CConnect(Input.Changed, function()
                if Input.UserInputState == Enum.UserInputState.End then
                    DragToggle = false
                end
            end))
        end
    end))

    AddConnection(CConnect(UI.InputChanged, function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseMovement or Input.UserInputType == Enum.UserInputType.Touch then
            DragInput = Input
        end
    end))

    AddConnection(CConnect(Services.UserInputService.InputChanged, function(Input)
        if Input == DragInput and DragToggle then
            UpdateInput(Input)
        end
    end))
end

Utils.Click = function(Object, Goal)
    local Hover = {[Goal] = Utils.MultColor3(Object[Goal], 0.95)}
    local Press = {[Goal] = Utils.MultColor3(Object[Goal], 1.1)}
    local Origin = {[Goal] = Object[Goal]}

    AddConnection(CConnect(Object.MouseEnter, function()
        Utils.Tween(Object, "Sine", "Out", .15, Hover)
    end))

    AddConnection(CConnect(Object.MouseLeave, function()
        Utils.Tween(Object, "Sine", "Out", .15, Origin)
    end))

    AddConnection(CConnect(Object.MouseButton1Down, function()
        Utils.Tween(Object, "Sine", "Out", .1, Press)
    end))

    AddConnection(CConnect(Object.MouseButton1Up, function()
        Utils.Tween(Object, "Sine", "Out", .15, Hover)
    end))
end

Utils.Hover = function(Object, Goal)
    local Hover = {[Goal] = Utils.MultColor3(Object[Goal], 0.95)}
    local Origin = {[Goal] = Object[Goal]}

    AddConnection(CConnect(Object.MouseEnter, function()
        Utils.Tween(Object, "Sine", "Out", .15, Hover)
    end))

    AddConnection(CConnect(Object.MouseLeave, function()
        Utils.Tween(Object, "Sine", "Out", .15, Origin)
    end))
end

Utils.TweenTrans = function(Object, Transparency)
    local Properties = {
        TextBox = "TextTransparency",
        TextLabel = "TextTransparency",
        TextButton = "TextTransparency",
        ImageButton = "ImageTransparency",
        ImageLabel = "ImageTransparency"
    }

    for _, Instance_ in ipairs(GetDescendants(Object)) do
        if IsA(Instance_, "GuiObject") then
            for Class, Property in pairs(Properties) do
                if IsA(Instance_, Class) and Instance_[Property] ~= 1 then
                    Utils.Tween(Instance_, "Sine", "Out", .3, {[Property] = Transparency})
                    break
                end
            end
            if Instance_.Name == "Overlay" and Transparency == 0 then
                Utils.Tween(Object, "Sine", "Out", .3, {BackgroundTransparency = .5})
            elseif Instance_.BackgroundTransparency ~= 1 then
                Utils.Tween(Instance_, "Sine", "Out", .3, {BackgroundTransparency = Transparency})
            end
        end
    end

    return Utils.Tween(Object, "Sine", "Out", .3, {BackgroundTransparency = Transparency})
end

Utils.Intro = function(Object)
    local Frame = Instancenew("Frame")
    local UICorner = Instancenew("UICorner")
    local UIGradient = Instancenew("UIGradient")
    local CornerRadius = FindFirstChild(Object, "UICorner") and Object.UICorner.CornerRadius or Theme.CornerRadius

    Frame.Name = "IntroFrame"
    Frame.ZIndex = 1000
    Frame.Size = UDim2.fromOffset(Object.AbsoluteSize.X, Object.AbsoluteSize.Y)
    Frame.AnchorPoint = Vector2new(.5, .5)
    Frame.Position = UDim2new(Object.Position.X.Scale, Object.Position.X.Offset + (Object.AbsoluteSize.X / 2), 
                             Object.Position.Y.Scale, Object.Position.Y.Offset + (Object.AbsoluteSize.Y / 2))
    Frame.BackgroundColor3 = Colors.Primary
    Frame.BorderSizePixel = 0

    UIGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Colors.Primary),
        ColorSequenceKeypoint.new(1, Colors.Secondary)
    })
    UIGradient.Rotation = 45
    UIGradient.Parent = Frame

    UICorner.CornerRadius = CornerRadius
    UICorner.Parent = Frame
    Frame.Parent = Object.Parent
    Utils.AddShadow(Frame)

    if Object.Visible then
        Frame.BackgroundTransparency = 1
        local Tween = Utils.Tween(Frame, "Sine", "Out", .25, {BackgroundTransparency = 0})
        CWait(Tween.Completed)
        Object.Visible = false
        Tween = Utils.Tween(Frame, "Sine", "Out", .25, {Size = UDim2.fromOffset(0, 0)})
        Utils.Tween(UICorner, "Sine", "Out", .25, {CornerRadius = UDimnew(1, 0)})
        CWait(Tween.Completed)
        Destroy(Frame)
    else
        Frame.Visible = true
        Frame.Size = UDim2.fromOffset(0, 0)
        UICorner.CornerRadius = UDimnew(1, 0)
        local Tween = Utils.Tween(Frame, "Sine", "Out", .25, {Size = UDim2.fromOffset(Object.AbsoluteSize.X, Object.AbsoluteSize.Y)})
        Utils.Tween(UICorner, "Sine", "Out", .25, {CornerRadius = CornerRadius})
        CWait(Tween.Completed)
        Object.Visible = true
        Tween = Utils.Tween(Frame, "Sine", "Out", .25, {BackgroundTransparency = 1})
        CWait(Tween.Completed)
        Destroy(Frame)
    end
end

Utils.MakeGradient = function(ColorTable)
    local Table = {}
    for Time, Color in pairs(ColorTable) do
        Table[#Table + 1] = ColorSequenceKeypoint.new(Time - 1, Color)
    end
    return ColorSequence.new(Table)
end

local UILibrary = {}
UILibrary.__index = UILibrary

UILibrary.new = function(ColorTheme)
    assert(typeof(ColorTheme) == "Color3", "[HELLWARE UI] ColorTheme must be a Color3.")
    local NewUI = {}
    local UI = Instancenew("ScreenGui")
    UI.Name = "HELLWARE_UI"
    UI.IgnoreGuiInset = true
    UI.ResetOnSpawn = false
    setmetatable(NewUI, UILibrary)
    NewUI.UI = UI
    NewUI.ColorTheme = ColorTheme
    NewUI.Connections = {}
    
    return NewUI
end

function UILibrary:SetTheme(NewTheme)
    for key, value in pairs(NewTheme) do
        if Colors[key] then
            Colors[key] = value
        end
    end
end

function UILibrary:LoadWindow(Title, Size)
    local Window = Clone(GuiObjects.Load.Window)
    local Main = Window.Main
    local Overlay = Main.Overlay
    local OverlayMain = Overlay.Main
    local ColorPicker = OverlayMain.ColorPicker
    local Settings = OverlayMain.Settings
    local ClosePicker = OverlayMain.Close
    local ColorCanvas = ColorPicker.ColorCanvas
    local ColorSlider = ColorPicker.ColorSlider
    local ColorGradient = ColorCanvas.ColorGradient
    local DarkGradient = ColorGradient.DarkGradient
    local CanvasBar = ColorGradient.Bar
    local RainbowGradient = ColorSlider.RainbowGradient
    local SliderBar = RainbowGradient.Bar
    local CanvasHitbox = ColorCanvas.Hitbox
    local SliderHitbox = ColorSlider.Hitbox
    local ColorPreview = Settings.ColorPreview
    local ColorOptions = Settings.Options
    local RedTextBox = ColorOptions.Red.TextBox
    local BlueTextBox = ColorOptions.Blue.TextBox
    local GreenTextBox = ColorOptions.Green.TextBox
    local RainbowToggle = ColorOptions.Rainbow
    
    local UICorner = Instancenew("UICorner")
    UICorner.CornerRadius = Theme.CornerRadius
    UICorner.Parent = Main
    Utils.AddShadow(Main)

    Utils.Click(OverlayMain.Close, "BackgroundColor3")
    Window.Size = Size
    Window.Position = UDim2new(0.5, -Size.X.Offset / 2, 0.5, -Size.Y.Offset / 2)
    Window.Main.Title.Text = "<b>HELLWARE</b> | " .. Title
    Window.Main.Title.RichText = true
    Window.Main.BackgroundColor3 = Colors.Background
    Window.Parent = self.UI

    Utils.Draggable(Window)

    local Idle = false
    local LeftWindow = false
    local Timer = tick()
    
    AddConnection(CConnect(Window.MouseEnter, function()
        LeftWindow = false
        if Idle then
            Idle = false
            Utils.TweenTrans(Window, 0)
        end
    end))
    
    AddConnection(CConnect(Window.MouseLeave, function()
        LeftWindow = true
        Timer = tick()
    end))

    AddConnection(CConnect(Services.RunService.RenderStepped, function()
        if LeftWindow then
            local Time = tick() - Timer
            if Time >= 3 and not Idle then
                Utils.TweenTrans(Window, .75)
                Idle = true
            end
        end
    end))

    local WindowLibrary = {}
    local PageCount = 0
    local SelectedPage

    WindowLibrary.GetPosition = function()
        return Window.Position
    end
    
    WindowLibrary.SetPosition = function(NewPos)
        Window.Position = NewPos
    end

    function WindowLibrary.NewPage(Title)
        local Page = Clone(GuiObjects.New.Page)
        local TextButton = Clone(GuiObjects.New.TextButton)
        local ButtonUICorner = Instancenew("UICorner")
        ButtonUICorner.CornerRadius = Theme.CornerRadius
        ButtonUICorner.Parent = TextButton

        if PageCount == 0 then
            TextButton.TextColor3 = Colors.TextPressed
            TextButton.BackgroundColor3 = Colors.Element
            TextButton.BorderColor3 = Colors.Border
            SelectedPage = Page
        end

        AddConnection(CConnect(TextButton.MouseEnter, function()
            if SelectedPage.Name ~= TextButton.Name then
                Utils.Tween(TextButton, "Sine", "Out", .15, {
                    TextColor3 = Colors.TextHover,
                    BackgroundColor3 = Utils.MultColor3(Colors.Element, 1.1),
                    BorderColor3 = Colors.Border
                })
            end
        end))

        AddConnection(CConnect(TextButton.MouseLeave, function()
            if SelectedPage.Name ~= TextButton.Name then
                Utils.Tween(TextButton, "Sine", "Out", .15, {
                    TextColor3 = Colors.Text,
                    BackgroundColor3 = Colors.Element,
                    BorderColor3 = Colors.Border
                })
            end
        end))

        AddConnection(CConnect(TextButton.MouseButton1Down, function()
            if SelectedPage.Name ~= TextButton.Name then
                Utils.Tween(TextButton, "Sine", "Out", .1, {
                    TextColor3 = Colors.TextPressed
                })
            end
        end))

        AddConnection(CConnect(TextButton.MouseButton1Click, function()
            if SelectedPage.Name ~= TextButton.Name then
                Utils.Tween(TextButton, "Sine", "Out", .15, {
                    TextColor3 = Colors.TextPressed,
                    BackgroundColor3 = Colors.Element,
                    BorderColor3 = Colors.Border
                })

                if Window.Main.Selection[SelectedPage.Name] then
                    Utils.Tween(Window.Main.Selection[SelectedPage.Name], "Sine", "Out", .15, {
                        TextColor3 = Colors.Text,
                        BackgroundColor3 = Colors.Element,
                        BorderColor3 = Colors.Border
                    })
                end

                SelectedPage = Page
                Window.Main.Container.UIPageLayout:JumpTo(SelectedPage)
            end
        end))

        Page.Name = Title
        TextButton.Name = Title
        TextButton.Text = Title
        TextButton.BackgroundColor3 = Colors.Element
        TextButton.TextColor3 = Colors.Text

        Page.Parent = Window.Main.Container
        TextButton.Parent = Window.Main.Selection

        PageCount = PageCount + 1

        local PageLibrary = {}

        function PageLibrary.NewSection(Title)
            local Section = GuiObjects.Section.Container:Clone()
            local SectionOptions = Section.Options
            local SectionUIListLayout = Section.Options.UIListLayout
            local SectionUICorner = Instancenew("UICorner")
            
            SectionUICorner.CornerRadius = Theme.CornerRadius
            SectionUICorner.Parent = Section
            Section.BackgroundColor3 = Colors.Background
            Section.Title.TextColor3 = Colors.Text
            Utils.AddShadow(Section)

            Utils.SmoothScroll(Section.Options, Theme.ScrollSmoothing)
            Section.Title.Text = Title
            Section.Parent = Page.Selection

            AddConnection(CConnect(GetPropertyChangedSignal(SectionUIListLayout, "AbsoluteContentSize"), function()
                SectionOptions.CanvasSize = UDim2.fromOffset(0, SectionUIListLayout.AbsoluteContentSize.Y + 10)
            end))

            local ElementLibrary = {}

            local function ToggleFunction(Container, Enabled, Callback)
                local Switch = Container.Switch
                local Hitbox = Container.Hitbox
                local UICorner = Instancenew("UICorner")
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = Container
                
                Container.BackgroundColor3 = Enabled and Colors.Primary or Colors.Element
                Switch.Position = Enabled and UDim2new(1, -18, 0, 2) or UDim2.fromOffset(2, 2)

                AddConnection(CConnect(Hitbox.MouseButton1Click, function()
                    Enabled = not Enabled
                    Utils.Tween(Switch, "Sine", "Out", .15, {
                        Position = Enabled and UDim2new(1, -18, 0, 2) or UDim2.fromOffset(2, 2)
                    })
                    Utils.Tween(Container, "Sine", "Out", .15, {
                        BackgroundColor3 = Enabled and Colors.Primary or Colors.Element
                    })
                    Callback(Enabled)
                end))
            end

            function ElementLibrary.Toggle(Title, Enabled, Callback)
                local Toggle = Clone(GuiObjects.Elements.Toggle)
                local Container = Toggle.Container
                ToggleFunction(Container, Enabled, Callback)
                Toggle.Title.Text = Title
                Toggle.Title.TextColor3 = Colors.Text
                Toggle.BackgroundColor3 = Colors.Background
                Toggle.Parent = Section.Options
            end

            function ElementLibrary.Slider(Title, Args, Callback)
                local Slider = Clone(GuiObjects.Elements.Slider)
                local Container = Slider.Container
                local ContainerSliderBar = Container.SliderBar
                local BarFrame = ContainerSliderBar.BarFrame
                local Bar = BarFrame.Bar
                local Label = Bar.Label
                local Hitbox = Container.Hitbox
                local UICorner = Instancenew("UICorner")
                
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = Container
                Bar.BackgroundColor3 = Colors.Primary
                Bar.Size = UDim2.fromScale(Args.Default / Args.Max, 1)
                Label.Text = tostring(Args.Default)
                Label.BackgroundTransparency = 0.8
                Label.TextTransparency = 0
                Label.TextColor3 = Colors.Text
                Container.Min.Text = tostring(Args.Min)
                Container.Max.Text = tostring(Args.Max)
                Container.Min.TextColor3 = Colors.Text
                Container.Max.TextColor3 = Colors.Text
                Slider.Title.Text = Title
                Slider.Title.TextColor3 = Colors.Text
                Slider.BackgroundColor3 = Colors.Background

                local Moving = false

                local function Update()
                    local RightBound = BarFrame.AbsoluteSize.X
                    local Position = clamp(Mouse.X - BarFrame.AbsolutePosition.X, 0, RightBound)
                    local Value = Args.Min + (Args.Max - Args.Min) * (Position / RightBound)
                    Value = floor(Value / Args.Step) * Args.Step
                    Callback(Value)

                    local Percent = (Value - Args.Min) / (Args.Max - Args.Min)
                    local Size = UDim2.fromScale(Percent, 1)
                    Utils.Tween(Bar, "Sine", "Out", .1, {Size = Size})
                    Label.Text = string.format("%.2f", Value)
                end

                AddConnection(CConnect(Hitbox.MouseButton1Down, function()
                    Moving = true
                    Utils.Tween(Label, "Sine", "Out", .15, {BackgroundTransparency = 0.5})
                    Update()
                end))

                AddConnection(CConnect(Services.UserInputService.InputEnded, function(Input)
                    if Input.UserInputType == Enum.UserInputType.MouseButton1 and Moving then
                        Moving = false
                        Utils.Tween(Label, "Sine", "Out", .15, {BackgroundTransparency = 0.8})
                    end
                end))

                AddConnection(CConnect(Mouse.Move, Debounce(function()
                    if Moving then
                        Update()
                    end
                end)))

                Slider.Parent = Section.Options
            end

            function ElementLibrary.ColorPicker(Title, DefaultColor, Callback)
                local SelectColor = Clone(GuiObjects.Elements.SelectColor)
                local CurrentColor = DefaultColor
                local Button = SelectColor.Button
                local H, S, V = ToHSV(DefaultColor)
                local Opened = false
                local Rainbow = false
                local UICorner = Instancenew("UICorner")
                
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = SelectColor
                SelectColor.BackgroundColor3 = Colors.Background
                SelectColor.Title.TextColor3 = Colors.Text

                local function UpdateText()
                    RedTextBox.PlaceholderText = tostring(floor(CurrentColor.R * 255))
                    GreenTextBox.PlaceholderText = tostring(floor(CurrentColor.G * 255))
                    BlueTextBox.PlaceholderText = tostring(floor(CurrentColor.B * 255))
                    RedTextBox.TextColor3 = Colors.Text
                    GreenTextBox.TextColor3 = Colors.Text
                    BlueTextBox.TextColor3 = Colors.Text
                end

                local function UpdateColor()
                    H, S, V = ToHSV(CurrentColor)
                    SliderBar.Position = UDim2new(0, 0, H, 2)
                    CanvasBar.Position = UDim2new(S, 2, 1 - V, 2)
                    ColorGradient.UIGradient.Color = Utils.MakeGradient({
                        [1] = Color3new(1, 1, 1),
                        [2] = Color3fromHSV(H, 1, 1)
                    })
                    ColorPreview.BackgroundColor3 = CurrentColor
                    UpdateText()
                end

                local function UpdateHue(Hue)
                    SliderBar.Position = UDim2new(0, 0, Hue, 2)
                    ColorGradient.UIGradient.Color = Utils.MakeGradient({
                        [1] = Color3new(1, 1, 1),
                        [2] = Color3fromHSV(Hue, 1, 1)
                    })
                    ColorPreview.BackgroundColor3 = CurrentColor
                    UpdateText()
                end

                local function ColorSliderInit()
                    local Moving = false
                    local function Update()
                        if Opened and not Rainbow then
                            local LowerBound = SliderHitbox.AbsoluteSize.Y
                            local Position = clamp(Mouse.Y - SliderHitbox.AbsolutePosition.Y, 0, LowerBound)
                            local Value = Position / LowerBound
                            H = Value
                            CurrentColor = Color3fromHSV(H, S, V)
                            ColorPreview.BackgroundColor3 = CurrentColor
                            ColorGradient.UIGradient.Color = Utils.MakeGradient({
                                [1] = Color3new(1, 1, 1),
                                [2] = Color3fromHSV(H, 1, 1)
                            })
                            UpdateText()
                            local Position = UDim2new(0, 0, Value, 2)
                            Utils.Tween(SliderBar, "Sine", "Out", .1, {Position = Position})
                            Callback(CurrentColor)
                        end
                    end

                    AddConnection(CConnect(SliderHitbox.MouseButton1Down, function()
                        Moving = true
                        Update()
                    end))

                    AddConnection(CConnect(Services.UserInputService.InputEnded, function(Input)
                        if Input.UserInputType == Enum.UserInputType.MouseButton1 and Moving then
                            Moving = false
                        end
                    end))

                    AddConnection(CConnect(Mouse.Move, Debounce(function()
                        if Moving then
                            Update()
                        end
                    end)))
                end

                local function ColorCanvasInit()
                    local Moving = false
                    local function Update()
                        if Opened then
                            local LowerBound = CanvasHitbox.AbsoluteSize.Y
                            local YPosition = clamp(Mouse.Y - CanvasHitbox.AbsolutePosition.Y, 0, LowerBound)
                            local YValue = YPosition / LowerBound
                            local RightBound = CanvasHitbox.AbsoluteSize.X
                            local XPosition = clamp(Mouse.X - CanvasHitbox.AbsolutePosition.X, 0, RightBound)
                            local XValue = XPosition / RightBound
                            S = XValue
                            V = 1 - YValue
                            CurrentColor = Color3fromHSV(H, S, V)
                            ColorPreview.BackgroundColor3 = CurrentColor
                            UpdateText()
                            local Position = UDim2new(XValue, 2, YValue, 2)
                            Utils.Tween(CanvasBar, "Sine", "Out", .1, {Position = Position})
                            Callback(CurrentColor)
                        end
                    end

                    AddConnection(CConnect(CanvasHitbox.MouseButton1Down, function()
                        Moving = true
                        Update()
                    end))

                    AddConnection(CConnect(Services.UserInputService.InputEnded, function(Input)
                        if Input.UserInputType == Enum.UserInputType.MouseButton1 and Moving then
                            Moving = false
                        end
                    end))

                    AddConnection(CConnect(Mouse.Move, Debounce(function()
                        if Moving then
                            Update()
                        end
                    end)))
                end

                ColorSliderInit()
                ColorCanvasInit()

                AddConnection(CConnect(Button.MouseButton1Click, function()
                    if not Opened then
                        Opened = true
                        UpdateColor()
                        RainbowToggle.Container.Switch.Position = Rainbow and UDim2new(1, -18, 0, 2) or UDim2.fromOffset(2, 2)
                        RainbowToggle.Container.BackgroundColor3 = Rainbow and Colors.Primary or Colors.Element
                        Overlay.Visible = true
                        OverlayMain.Visible = false
                        Utils.Intro(OverlayMain)
                    end
                end))

                AddConnection(CConnect(ClosePicker.MouseButton1Click, Debounce(function()
                    Button.BackgroundColor3 = CurrentColor
                    Utils.Intro(OverlayMain)
                    Overlay.Visible = false
                    Opened = false
                end)))

                AddConnection(CConnect(RedTextBox.FocusLost, function()
                    if Opened then
                        local Number = tonumber(RedTextBox.Text)
                        if Number then
                            Number = clamp(floor(Number), 0, 255)
                            CurrentColor = Color3new(Number / 255, CurrentColor.G, CurrentColor.B)
                            UpdateColor()
                            RedTextBox.PlaceholderText = tostring(Number)
                            Callback(CurrentColor)
                        end
                        RedTextBox.Text = ""
                    end
                end))

                AddConnection(CConnect(GreenTextBox.FocusLost, function()
                    if Opened then
                        local Number = tonumber(GreenTextBox.Text)
                        if Number then
                            Number = clamp(floor(Number), 0, 255)
                            CurrentColor = Color3new(CurrentColor.R, Number / 255, CurrentColor.B)
                            UpdateColor()
                            GreenTextBox.PlaceholderText = tostring(Number)
                            Callback(CurrentColor)
                        end
                        GreenTextBox.Text = ""
                    end
                end))

                AddConnection(CConnect(BlueTextBox.FocusLost, function()
                    if Opened then
                        local Number = tonumber(BlueTextBox.Text)
                        if Number then
                            Number = clamp(floor(Number), 0, 255)
                            CurrentColor = Color3new(CurrentColor.R, CurrentColor.G, Number / 255)
                            UpdateColor()
                            BlueTextBox.PlaceholderText = tostring(Number)
                            Callback(CurrentColor)
                        end
                        BlueTextBox.Text = ""
                    end
                end))

                ToggleFunction(RainbowToggle.Container, false, function(Callback)
                    if Opened then
                        Rainbow = Callback
                    end
                end)

                AddConnection(CConnect(Services.RunService.RenderStepped, function()
                    if Rainbow then
                        local Hue = (tick() / 5) % 1
                        CurrentColor = Color3fromHSV(Hue, S, V)
                        if Opened then
                            UpdateHue(Hue)
                        end
                        Button.BackgroundColor3 = CurrentColor
                        Callback(CurrentColor)
                    end
                end))

                Button.BackgroundColor3 = DefaultColor
                SelectColor.Title.Text = Title
                SelectColor.Parent = Section.Options
            end

            function ElementLibrary.Dropdown(Title, Options, Callback)
                local DropdownElement = GuiObjects.Elements.Dropdown.DropdownElement:Clone()
                local DropdownSelection = GuiObjects.Elements.Dropdown.DropdownSelection:Clone()
                local TextButton = GuiObjects.Elements.Dropdown.TextButton
                local Button = DropdownElement.Button
                local UICorner = Instancenew("UICorner")
                local SelectionUICorner = Instancenew("UICorner")
                
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = DropdownElement
                SelectionUICorner.CornerRadius = Theme.CornerRadius
                SelectionUICorner.Parent = DropdownSelection
                DropdownElement.BackgroundColor3 = Colors.Background
                DropdownElement.Title.TextColor3 = Colors.Text
                
                local Opened = false
                local Size = (TextButton.Size.Y.Offset + 5) * #Options

                local function ToggleDropdown()
                    Opened = not Opened
                    if Opened then
                        DropdownSelection.Frame.Visible = true
                        DropdownSelection.Visible = true
                        Utils.Tween(DropdownSelection, "Sine", "Out", .2, {Size = UDim2new(1, -10, 0, Size)})
                        Utils.Tween(DropdownElement.Button, "Sine", "Out", .2, {Rotation = 180})
                    else
                        Utils.Tween(DropdownElement.Button, "Sine", "Out", .2, {Rotation = 0})
                        CWait(Utils.Tween(DropdownSelection, "Sine", "Out", .2, {Size = UDim2new(1, -10, 0, 0)}).Completed)
                        DropdownSelection.Frame.Visible = false
                        DropdownSelection.Visible = false
                    end
                end

                for _, v in ipairs(Options) do
                    local Clone = Clone(TextButton)
                    local CloneUICorner = Instancenew("UICorner")
                    CloneUICorner.CornerRadius = Theme.CornerRadius
                    CloneUICorner.Parent = Clone
                    Clone.BackgroundColor3 = Colors.Element
                    Clone.TextColor3 = Colors.Text
                    
                    AddConnection(CConnect(Clone.MouseButton1Click, function()
                        DropdownElement.Title.Text = Title .. ": " .. v
                        Callback(v)
                        ToggleDropdown()
                    end))
                    Utils.Click(Clone, "BackgroundColor3")
                    Clone.Text = v
                    Clone.Parent = DropdownSelection.Container
                end

                AddConnection(CConnect(Button.MouseButton1Click, ToggleDropdown))
                DropdownElement.Title.Text = Title
                DropdownSelection.Visible = false
                DropdownSelection.Frame.Visible = false
                DropdownSelection.Size = UDim2new(1, -10, 0, 0)
                DropdownElement.Parent = Section.Options
                DropdownSelection.Parent = Section.Options
            end

            function ElementLibrary.Keybind(Title, DefaultKey, Callback)
                local Keybind = Clone(GuiObjects.Elements.Toggle)
                local Container = Keybind.Container
                local Hitbox = Container.Hitbox
                local Label = Container.Switch
                local UICorner = Instancenew("UICorner")
                
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = Keybind
                Keybind.BackgroundColor3 = Colors.Background
                Keybind.Title.TextColor3 = Colors.Text
                Label.TextColor3 = Colors.Text
                
                Label.Text = DefaultKey or "None"
                Keybind.Title.Text = Title
                
                local Binding = false
                local CurrentKey = DefaultKey
                
                AddConnection(CConnect(Hitbox.MouseButton1Click, function()
                    Binding = true
                    Label.Text = "Press a key..."
                    Utils.Tween(Label, "Sine", "Out", .15, {TextColor3 = Colors.TextHover})
                end))
                
                AddConnection(CConnect(Services.UserInputService.InputBegan, function(Input)
                    if Binding and Input.UserInputType == Enum.UserInputType.Keyboard then
                        CurrentKey = Input.KeyCode.Name
                        Label.Text = CurrentKey
                        Binding = false
                        Utils.Tween(Label, "Sine", "Out", .15, {TextColor3 = Colors.Text})
                        Callback(CurrentKey)
                    end
                end))
                
                Keybind.Parent = Section.Options
            end

            function ElementLibrary.Button(Title, Callback)
                local Button = Clone(GuiObjects.Elements.TextButton)
                local UICorner = Instancenew("UICorner")
                local UIGradient = Instancenew("UIGradient")
                
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = Button
                UIGradient.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Colors.Primary),
                    ColorSequenceKeypoint.new(1, Colors.Secondary)
                })
                UIGradient.Rotation = 45
                UIGradient.Parent = Button
                
                Button.Title.Text = Title
                Button.BackgroundColor3 = Colors.Element
                Button.Title.TextColor3 = Colors.Text
                
                AddConnection(CConnect(Button.MouseButton1Click, function()
                    Utils.Tween(Button, "Sine", "Out", .1, {BackgroundColor3 = Colors.Accent})
                    task.wait(0.1)
                    Utils.Tween(Button, "Sine", "Out", .1, {BackgroundColor3 = Colors.Element})
                    Callback()
                end))
                
                Utils.Click(Button, "BackgroundColor3")
                Button.Parent = Section.Options
            end

            function ElementLibrary.Label(Title)
                local Label = Clone(GuiObjects.Elements.TextLabel)
                local UICorner = Instancenew("UICorner")
                
                UICorner.CornerRadius = Theme.CornerRadius
                UICorner.Parent = Label
                Label.Title.Text = Title
                Label.BackgroundColor3 = Colors.Background
                Label.Title.TextColor3 = Colors.Text
                Label.Parent = Section.Options
            end

            return ElementLibrary
        end

        return PageLibrary
    end

    return WindowLibrary
end

print("HELLWARE UI Loaded...")
return UILibrary
