--[[
    OrionUI - Professional Roblox UI Library
    Inspired by the best features from top Roblox UI libraries
    
    API:
    Orion:Window({Name, SubName, Logo, Accent, Key})
    Window:Page({Name, Icon})
    Page:Section({Name, Side})
    Section:Toggle/Slider/Dropdown/Keybind/Colorpicker/Button/Textbox/Listbox
]]

local OrionUI = {}

-- Services
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Theme (customizable)
OrionUI.Theme = {
    Accent = Color3.fromRGB(0, 195, 255),
    AccentGradient = Color3.fromRGB(0, 116, 224),
    Background = Color3.fromRGB(12, 12, 14),
    Background2 = Color3.fromRGB(10, 10, 12),
    Text = Color3.fromRGB(235, 235, 235),
    SubText = Color3.fromRGB(160, 160, 160),
    Outline = Color3.fromRGB(25, 25, 28),
    Element = Color3.fromRGB(16, 16, 18),
    SectionBackground = Color3.fromRGB(10, 10, 12),
    SectionTop = Color3.fromRGB(28, 27, 31),
    Danger = Color3.fromRGB(255, 60, 60),
    Success = Color3.fromRGB(60, 255, 120)
}

OrionUI.Flags = {}
OrionUI.Fonts = {
    SemiBold = Font.new("rbxassetid://12187365364", Enum.FontWeight.SemiBold, Enum.FontStyle.Normal),
    Regular = Font.new("rbxassetid://12187365364", Enum.FontWeight.Regular, Enum.FontStyle.Normal),
    Light = Font.new("rbxassetid://12187365364", Enum.FontWeight.Light, Enum.FontStyle.Normal)
}
OrionUI.Font = OrionUI.Fonts.SemiBold

-- Utility Functions
local function CreateInstance(class, props)
    local inst = Instance.new(class)
    for k, v in pairs(props) do
        inst[k] = v
    end
    return inst
end

local function Round(instance, radius)
    local corner = CreateInstance("UICorner", {
        CornerRadius = UDim.new(0, radius),
        Parent = instance
    })
    return corner
end

local function AddDraggable(frame)
    local dragging = false
    local dragInput = nil
    local dragStart = nil
    local startPos = nil

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            
            local connection
            connection = input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    connection:Disconnect()
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Window Creation
function OrionUI:Window(options)
    options = options or {}
    local Window = {
        Name = options.Name or "OrionUI",
        SubName = options.SubName or "Premium UI Library",
        Logo = options.Logo or "rbxassetid://120959262762131",
        Accent = options.Accent or OrionUI.Theme.Accent,
        Key = options.Key or Enum.KeyCode.RightControl,
        Pages = {},
        IsOpen = true
    }

    -- Holder
    local ScreenGui = CreateInstance("ScreenGui", {
        Name = "OrionUI",
        Parent = game:GetService("CoreGui"),
        ZIndexBehavior = Enum.ZIndexBehavior.Global,
        DisplayOrder = 9999,
        ResetOnSpawn = false
    })

    -- Main Frame
    local MainFrame = CreateInstance("Frame", {
        Parent = ScreenGui,
        BackgroundColor3 = OrionUI.Theme.Background,
        BorderSizePixel = 0,
        Position = UDim2.new(0.5, -350, 0.5, -300),
        Size = UDim2.new(0, 700, 0, 600),
        ClipsDescendants = true
    })
    
    local MainFrameOutline = CreateInstance("Frame", {
        Parent = MainFrame,
        BackgroundColor3 = OrionUI.Theme.Outline,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 1, 0, 1),
        Size = UDim2.new(1, -2, 1, -2)
    })

    -- Background Pattern
    local BGPattern = CreateInstance("ImageLabel", {
        Parent = MainFrameOutline,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 1, 0),
        Image = "rbxassetid://15453092054",
        ImageColor3 = Color3.fromRGB(8, 8, 10),
        ImageTransparency = 0.5,
        ScaleType = Enum.ScaleType.Tile,
        TileSize = UDim2.new(0, 4, 0, 4)
    })

    -- Title Bar
    local TitleBar = CreateInstance("Frame", {
        Parent = MainFrameOutline,
        BackgroundColor3 = OrionUI.Theme.SectionTop,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 45),
        ZIndex = 10
    })

    local Logo = CreateInstance("ImageLabel", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 30, 0, 30),
        Position = UDim2.new(0, 12, 0.5, -15),
        Image = Window.Logo,
        ImageColor3 = Window.Accent,
        ZIndex = 11
    })

    local TitleText = CreateInstance("TextLabel", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 50, 0, 8),
        Size = UDim2.new(1, -60, 0, 18),
        Text = Window.Name,
        FontFace = OrionUI.Font,
        TextColor3 = OrionUI.Theme.Text,
        TextSize = 16,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 11
    })

    local SubTitleText = CreateInstance("TextLabel", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 50, 0, 24),
        Size = UDim2.new(1, -60, 0, 14),
        Text = Window.SubName,
        FontFace = OrionUI.Font,
        TextColor3 = OrionUI.Theme.SubText,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 11
    })

    -- Close Button
    local CloseBtn = CreateInstance("TextButton", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -40, 0.5, -10),
        Size = UDim2.new(0, 20, 0, 20),
        Text = "",
        AutoButtonColor = false,
        ZIndex = 11
    })

    local CloseIcon = CreateInstance("ImageLabel", {
        Parent = CloseBtn,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 1, 0),
        Image = "rbxassetid://6031091004",
        ImageColor3 = OrionUI.Theme.SubText,
        ZIndex = 12
    })

    CloseBtn.MouseButton1Click:Connect(function()
        Window.IsOpen = false
        MainFrameOutline:TweenPosition(UDim2.new(0, -1000, 0.5, 0), "Out", "Quad", 0.3, true)
    end)

    -- Tabs Container
    local TabsContainer = CreateInstance("Frame", {
        Parent = MainFrameOutline,
        BackgroundColor3 = OrionUI.Theme.Background2,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 0, 0, 45),
        Size = UDim2.new(0, 180, 1, -45),
        ZIndex = 5
    })

    local TabsList = CreateInstance("UIListLayout", {
        Parent = TabsContainer,
        Padding = UDim.new(0, 4),
        SortOrder = Enum.SortOrder.LayoutOrder
    })

    local TabsPadding = CreateInstance("UIPadding", {
        Parent = TabsContainer,
        PaddingTop = UDim.new(0, 10),
        PaddingLeft = UDim.new(0, 8),
        PaddingRight = UDim.new(0, 8)
    })

    -- Pages Container
    local PagesContainer = CreateInstance("Frame", {
        Parent = MainFrameOutline,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 185, 0, 50),
        Size = UDim2.new(1, -190, 1, -55),
        ZIndex = 3
    })

    -- Make draggable
    AddDraggable(MainFrame)

    -- Window Functions
    function Window:SetOpen(state)
        Window.IsOpen = state
        MainFrameOutline.Visible = state
    end

    function Window:SetCenter()
        MainFrame.Position = UDim2.new(0.5, -MainFrame.AbsoluteSize.X/2, 0.5, -MainFrame.AbsoluteSize.Y/2)
    end

    -- Page Creation
    function Window:Page(options)
        options = options or {}
        local Page = {
            Name = options.Name or "Page",
            Icon = options.Icon or "rbxassetid://15453302474",
            Window = self,
            Sections = {}
        }

        -- Tab Button
        local TabBtn = CreateInstance("TextButton", {
            Parent = TabsContainer,
            BackgroundColor3 = OrionUI.Theme.Element,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 0, 40),
            Text = "",
            AutoButtonColor = false,
            ZIndex = 6
        })
        Round(TabBtn, 6)

        local TabIcon = CreateInstance("ImageLabel", {
            Parent = TabBtn,
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 20, 0, 20),
            Position = UDim2.new(0, 8, 0.5, -10),
            Image = Page.Icon,
            ImageColor3 = OrionUI.Theme.SubText,
            ZIndex = 7
        })

        local TabText = CreateInstance("TextLabel", {
            Parent = TabBtn,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 35, 0, 0),
            Size = UDim2.new(1, -40, 1, 0),
            Text = Page.Name,
            FontFace = OrionUI.Font,
            TextColor3 = OrionUI.Theme.SubText,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            ZIndex = 7
        })

        -- Page Content
        local PageContent = CreateInstance("ScrollingFrame", {
            Parent = PagesContainer,
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 1, 0),
            CanvasSize = UDim2.new(0, 0, 0, 0),
            ScrollBarThickness = 3,
            ScrollBarImageColor3 = Window.Accent,
            Visible = false,
            ZIndex = 4
        })

        local PageList = CreateInstance("UIListLayout", {
            Parent = PageContent,
            Padding = UDim.new(0, 12),
            SortOrder = Enum.SortOrder.LayoutOrder
        })

        local PagePadding = CreateInstance("UIPadding", {
            Parent = PageContent,
            PaddingTop = UDim.new(0, 8),
            PaddingLeft = UDim.new(0, 8),
            PaddingRight = UDim.new(0, 8)
        })

        PageList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            PageContent.CanvasSize = UDim2.new(0, 0, 0, PageList.AbsoluteContentSize.Y + 20)
        end)

        -- Tab Logic
        local function Activate()
            for _, p in ipairs(Window.Pages) do
                p.Content.Visible = false
                p.TabBtn.BackgroundTransparency = 1
                p.TabIcon.ImageColor3 = OrionUI.Theme.SubText
                p.TabText.TextColor3 = OrionUI.Theme.SubText
            end
            PageContent.Visible = true
            TabBtn.BackgroundTransparency = 0
            TabIcon.ImageColor3 = Window.Accent
            TabText.TextColor3 = OrionUI.Theme.Text
        end

        TabBtn.MouseButton1Click:Connect(Activate)

        -- Store page
        Page.Content = PageContent
        Page.TabBtn = TabBtn
        Page.TabIcon = TabIcon
        Page.TabText = TabText

        table.insert(Window.Pages, Page)

        if #Window.Pages == 1 then
            Activate()
        end

        -- Section Creation
        function Page:Section(options)
            options = options or {}
            local Section = {
                Name = options.Name or "Section",
                Side = options.Side or "Left",
                Page = self,
                Window = Window
            }

            local SectionFrame = CreateInstance("Frame", {
                Parent = PageContent,
                BackgroundColor3 = OrionUI.Theme.SectionBackground,
                BorderSizePixel = 0,
                Size = UDim2.new(0.5, -6, 0, 200),
                ZIndex = 4
            })
            Round(SectionFrame, 8)

            local SectionInner = CreateInstance("Frame", {
                Parent = SectionFrame,
                BackgroundColor3 = OrionUI.Theme.Background2,
                BorderSizePixel = 0,
                Position = UDim2.new(0, 1, 0, 1),
                Size = UDim2.new(1, -2, 1, -2)
            })
            Round(SectionInner, 7)

            local SectionTitle = CreateInstance("TextLabel", {
                Parent = SectionInner,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 12, 0, 8),
                Size = UDim2.new(1, -24, 0, 16),
                Text = Section.Name,
                FontFace = OrionUI.Font,
                TextColor3 = OrionUI.Theme.SubText,
                TextSize = 12,
                TextXAlignment = Enum.TextXAlignment.Left,
                ZIndex = 5
            })

            local SectionContent = CreateInstance("Frame", {
                Parent = SectionInner,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 4, 0, 30),
                Size = UDim2.new(1, -8, 1, -34),
                ZIndex = 4
            })

            local SectionList = CreateInstance("UIListLayout", {
                Parent = SectionContent,
                Padding = UDim.new(0, 6),
                SortOrder = Enum.SortOrder.LayoutOrder
            })

            local SectionPadding = CreateInstance("UIPadding", {
                Parent = SectionContent,
                PaddingBottom = UDim.new(0, 4)
            })

            -- Toggle
            function Section:Toggle(options)
                options = options or {}
                local Toggle = {
                    Name = options.Name or "Toggle",
                    Flag = options.Flag,
                    Default = options.Default or false,
                    Callback = options.Callback or function() end,
                    State = false
                }
                Toggle.State = Toggle.Default

                local ToggleFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 28),
                    ZIndex = 5
                })

                local ToggleLabel = CreateInstance("TextLabel", {
                    Parent = ToggleFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -40, 1, 0),
                    Text = Toggle.Name,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 13,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local ToggleBox = CreateInstance("Frame", {
                    Parent = ToggleFrame,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Position = UDim2.new(1, 0, 0.5, 0),
                    Size = UDim2.new(0, 18, 0, 18),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    ZIndex = 6
                })
                Round(ToggleBox, 4)

                local ToggleDot = CreateInstance("Frame", {
                    Parent = ToggleBox,
                    AnchorPoint = Vector2.new(0.5, 0.5),
                    Position = UDim2.new(0.5, 0, 0.5, 0),
                    Size = UDim2.new(0, 0, 0, 0),
                    BackgroundColor3 = Window.Accent,
                    BorderSizePixel = 0,
                    ZIndex = 7
                })
                Round(ToggleDot, 3)

                local ToggleBtn = CreateInstance("TextButton", {
                    Parent = ToggleFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 1, 0),
                    Text = "",
                    ZIndex = 10
                })

                local function UpdateToggle()
                    if Toggle.State then
                        ToggleDot:TweenSize(UDim2.new(0, 12, 0, 12), "Out", "Quad", 0.15)
                        ToggleLabel.TextColor3 = Window.Accent
                    else
                        ToggleDot:TweenSize(UDim2.new(0, 0, 0, 0), "Out", "Quad", 0.15)
                        ToggleLabel.TextColor3 = OrionUI.Theme.Text
                    end
                    Toggle.Callback(Toggle.State)
                    if Toggle.Flag then OrionUI.Flags[Toggle.Flag] = Toggle.State end
                end

                ToggleBtn.MouseButton1Click:Connect(function()
                    Toggle.State = not Toggle.State
                    UpdateToggle()
                end)

                function Toggle:Set(state)
                    Toggle.State = state
                    UpdateToggle()
                end

                function Toggle:Get()
                    return Toggle.State
                end

                if Toggle.Default then UpdateToggle() end

                return Toggle
            end

            -- Slider
            function Section:Slider(options)
                options = options or {}
                local Slider = {
                    Name = options.Name or "Slider",
                    Flag = options.Flag,
                    Min = options.Min or 0,
                    Max = options.Max or 100,
                    Default = options.Default or 50,
                    Suffix = options.Suffix or "",
                    Decimals = options.Decimals or 0,
                    Callback = options.Callback or function() end,
                    Value = 0
                }
                Slider.Value = Slider.Default

                local SliderFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 40),
                    ZIndex = 5
                })

                local SliderLabel = CreateInstance("TextLabel", {
                    Parent = SliderFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -50, 0, 18),
                    Text = Slider.Name,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 13,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local SliderValue = CreateInstance("TextLabel", {
                    Parent = SliderFrame,
                    AnchorPoint = Vector2.new(1, 0),
                    Position = UDim2.new(1, 0, 0, 0),
                    Size = UDim2.new(0, 45, 0, 18),
                    Text = tostring(Slider.Value) .. Slider.Suffix,
                    FontFace = OrionUI.Font,
                    TextColor3 = Window.Accent,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Right,
                    ZIndex = 6
                })

                local SliderTrack = CreateInstance("Frame", {
                    Parent = SliderFrame,
                    Position = UDim2.new(0, 0, 0, 24),
                    Size = UDim2.new(1, 0, 0, 6),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    ZIndex = 6
                })
                Round(SliderTrack, 3)

                local SliderFill = CreateInstance("Frame", {
                    Parent = SliderTrack,
                    BackgroundColor3 = Window.Accent,
                    BorderSizePixel = 0,
                    Size = UDim2.new(0.5, 0, 1, 0),
                    ZIndex = 7
                })
                Round(SliderFill, 3)

                local SliderBtn = CreateInstance("TextButton", {
                    Parent = SliderFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 1, 0),
                    Text = "",
                    ZIndex = 10
                })

                local dragging = false

                local function UpdateSlider(input)
                    local x = math.clamp((input.Position.X - SliderTrack.AbsolutePosition.X) / SliderTrack.AbsoluteSize.X, 0, 1)
                    local val = math.round((Slider.Min + (Slider.Max - Slider.Min) * x) * (10 ^ Slider.Decimals)) / (10 ^ Slider.Decimals)
                    Slider.Value = val
                    SliderFill.Size = UDim2.new(x, 0, 1, 0)
                    SliderValue.Text = tostring(val) .. Slider.Suffix
                    Slider.Callback(val)
                    if Slider.Flag then OrionUI.Flags[Slider.Flag] = val end
                end

                SliderBtn.InputBegan:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = true
                        UpdateSlider(input)
                    end
                end)

                SliderBtn.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = false
                    end
                end)

                UserInputService.InputChanged:Connect(function(input)
                    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                        UpdateSlider(input)
                    end
                end)

                function Slider:Set(value)
                    Slider.Value = math.clamp(value, Slider.Min, Slider.Max)
                    local x = (Slider.Value - Slider.Min) / (Slider.Max - Slider.Min)
                    SliderFill.Size = UDim2.new(x, 0, 1, 0)
                    SliderValue.Text = tostring(Slider.Value) .. Slider.Suffix
                    Slider.Callback(Slider.Value)
                    if Slider.Flag then OrionUI.Flags[Slider.Flag] = Slider.Value end
                end

                function Slider:Get()
                    return Slider.Value
                end

                -- Set default
                local defaultX = (Slider.Default - Slider.Min) / (Slider.Max - Slider.Min)
                SliderFill.Size = UDim2.new(defaultX, 0, 1, 0)
                SliderValue.Text = tostring(Slider.Default) .. Slider.Suffix

                return Slider
            end

            -- Dropdown
            function Section:Dropdown(options)
                options = options or {}
                local Dropdown = {
                    Name = options.Name or "Dropdown",
                    Flag = options.Flag,
                    Items = options.Items or {"Option 1", "Option 2", "Option 3"},
                    Default = options.Default or 1,
                    Callback = options.Callback or function() end,
                    Value = "",
                    Index = 1
                }
                Dropdown.Index = Dropdown.Default
                Dropdown.Value = Dropdown.Items[Dropdown.Index]

                local DropdownFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 32),
                    ClipsDescendants = true,
                    ZIndex = 5
                })

                local DropdownLabel = CreateInstance("TextLabel", {
                    Parent = DropdownFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 16),
                    Text = Dropdown.Name,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 13,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local DropdownBtn = CreateInstance("TextButton", {
                    Parent = DropdownFrame,
                    Position = UDim2.new(0, 0, 0, 20),
                    Size = UDim2.new(1, 0, 0, 26),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    Text = "  " .. Dropdown.Value,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    AutoButtonColor = false,
                    ZIndex = 6
                })
                Round(DropdownBtn, 4)

                local DropdownIcon = CreateInstance("ImageLabel", {
                    Parent = DropdownBtn,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Position = UDim2.new(1, -8, 0.5, 0),
                    Size = UDim2.new(0, 12, 0, 12),
                    Image = "rbxassetid://6034818379",
                    ImageColor3 = OrionUI.Theme.SubText,
                    BackgroundTransparency = 1,
                    ZIndex = 7
                })

                local DropdownList = CreateInstance("Frame", {
                    Parent = DropdownFrame,
                    Position = UDim2.new(0, 0, 0, 48),
                    Size = UDim2.new(1, 0, 0, 0),
                    BackgroundColor3 = OrionUI.Theme.Background2,
                    BorderSizePixel = 0,
                    AutomaticSize = Enum.AutomaticSize.Y,
                    Visible = false,
                    ZIndex = 20
                })

                local DropdownListInner = CreateInstance("Frame", {
                    Parent = DropdownList,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 0),
                    AutomaticSize = Enum.AutomaticSize.Y,
                    ZIndex = 21
                })

                local DropdownListLayout = CreateInstance("UIListLayout", {
                    Parent = DropdownListInner,
                    Padding = UDim.new(0, 2),
                    SortOrder = Enum.SortOrder.LayoutOrder
                })

                local DropdownListPadding = CreateInstance("UIPadding", {
                    Parent = DropdownListInner,
                    PaddingTop = UDim.new(0, 4),
                    PaddingBottom = UDim.new(0, 4)
                })

                -- Create options
                for i, item in ipairs(Dropdown.Items) do
                    local OptionBtn = CreateInstance("TextButton", {
                        Parent = DropdownListInner,
                        BackgroundTransparency = 1,
                        Size = UDim2.new(1, 0, 0, 24),
                        Text = "  " .. item,
                        FontFace = OrionUI.Font,
                        TextColor3 = OrionUI.Theme.SubText,
                        TextSize = 12,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        AutoButtonColor = false,
                        ZIndex = 22
                    })

                    OptionBtn.MouseButton1Click:Connect(function()
                        Dropdown.Index = i
                        Dropdown.Value = item
                        DropdownBtn.Text = "  " .. item
                        DropdownList.Visible = false
                        DropdownIcon:TweenRotation(0, "Out", "Quad", 0.2)
                        Dropdown.Callback(item)
                        if Dropdown.Flag then OrionUI.Flags[Dropdown.Flag] = item end
                    end)

                    OptionBtn.MouseEnter:Connect(function()
                        OptionBtn.TextColor3 = Window.Accent
                    end)

                    OptionBtn.MouseLeave:Connect(function()
                        OptionBtn.TextColor3 = OrionUI.Theme.SubText
                    end)
                end

                local isOpen = false

                DropdownBtn.MouseButton1Click:Connect(function()
                    isOpen = not isOpen
                    DropdownList.Visible = isOpen
                    if isOpen then
                        DropdownIcon:TweenRotation(180, "Out", "Quad", 0.2)
                    else
                        DropdownIcon:TweenRotation(0, "Out", "Quad", 0.2)
                    end
                end)

                function Dropdown:Set(value)
                    for i, item in ipairs(Dropdown.Items) do
                        if item == value then
                            Dropdown.Index = i
                            Dropdown.Value = item
                            DropdownBtn.Text = "  " .. item
                            Dropdown.Callback(item)
                            if Dropdown.Flag then OrionUI.Flags[Dropdown.Flag] = item end
                            break
                        end
                    end
                end

                function Dropdown:Get()
                    return Dropdown.Value
                end

                return Dropdown
            end

            -- Button
            function Section:Button(options)
                options = options or {}
                local Button = {
                    Name = options.Name or "Button",
                    Callback = options.Callback or function() end
                }

                local ButtonFrame = CreateInstance("TextButton", {
                    Parent = SectionContent,
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    Size = UDim2.new(1, 0, 0, 32),
                    Text = Button.Name,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 13,
                    AutoButtonColor = false,
                    ZIndex = 5
                })
                Round(ButtonFrame, 4)

                ButtonFrame.MouseButton1Click:Connect(function()
                    Button.Callback()
                end)

                ButtonFrame.MouseEnter:Connect(function()
                    ButtonFrame.BackgroundColor3 = Window.Accent
                end)

                ButtonFrame.MouseLeave:Connect(function()
                    ButtonFrame.BackgroundColor3 = OrionUI.Theme.Element
                end)

                return Button
            end

            -- Keybind
            function Section:Keybind(options)
                options = options or {}
                local Keybind = {
                    Name = options.Name or "Keybind",
                    Flag = options.Flag,
                    Default = options.Default or Enum.KeyCode.X,
                    Mode = options.Mode or "Toggle",
                    Callback = options.Callback or function() end,
                    Key = nil,
                    Active = false
                }
                Keybind.Key = Keybind.Default

                local KeybindFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 28),
                    ZIndex = 5
                })

                local KeybindLabel = CreateInstance("TextLabel", {
                    Parent = KeybindFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -70, 1, 0),
                    Text = Keybind.Name,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 13,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local KeybindValue = CreateInstance("TextLabel", {
                    Parent = KeybindFrame,
                    AnchorPoint = Vector2.new(1, 0),
                    Position = UDim2.new(1, 0, 0, 0),
                    Size = UDim2.new(0, 65, 1, 0),
                    Text = "[" .. string.sub(tostring(Keybind.Key.Name), -4) .. "]",
                    FontFace = OrionUI.Font,
                    TextColor3 = Window.Accent,
                    TextSize = 11,
                    TextXAlignment = Enum.TextXAlignment.Right,
                    ZIndex = 6
                })

                local KeybindBtn = CreateInstance("TextButton", {
                    Parent = KeybindFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 1, 0),
                    Text = "",
                    ZIndex = 10
                })

                local listening = false

                KeybindBtn.MouseButton1Click:Connect(function()
                    listening = true
                    KeybindValue.Text = "[...]"
                    KeybindValue.TextColor3 = OrionUI.Theme.Danger
                end)

                UserInputService.InputBegan:Connect(function(input)
                    if listening then
                        if input.UserInputType == Enum.UserInputType.Keyboard then
                            Keybind.Key = input.KeyCode
                            KeybindValue.Text = "[" .. string.sub(tostring(input.KeyCode.Name), -4) .. "]"
                            KeybindValue.TextColor3 = Window.Accent
                            listening = false
                        end
                    else
                        if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Keybind.Key then
                            if Keybind.Mode == "Toggle" then
                                Keybind.Active = not Keybind.Active
                                Keybind.Callback(Keybind.Active)
                            elseif Keybind.Mode == "Hold" then
                                Keybind.Active = true
                                Keybind.Callback(true)
                            end
                        end
                    end
                end)

                UserInputService.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Keybind.Key and Keybind.Mode == "Hold" then
                        Keybind.Active = false
                        Keybind.Callback(false)
                    end
                end)

                function Keybind:Set(key)
                    Keybind.Key = key
                    KeybindValue.Text = "[" .. string.sub(tostring(key.Name), -4) .. "]"
                end

                function Keybind:Get()
                    return {Key = Keybind.Key, Mode = Keybind.Mode, Active = Keybind.Active}
                end

                return Keybind
            end

            -- Colorpicker
            function Section:Colorpicker(options)
                options = options or {}
                local Colorpicker = {
                    Name = options.Name or "Colorpicker",
                    Flag = options.Flag,
                    Default = options.Default or Color3.fromRGB(255, 255, 255),
                    Callback = options.Callback or function() end,
                    Color = Colorpicker.Default,
                    Alpha = 0
                }
                Colorpicker.Color = Colorpicker.Default

                local ColorpickerFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 28),
                    ZIndex = 5
                })

                local ColorpickerLabel = CreateInstance("TextLabel", {
                    Parent = ColorpickerFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -40, 1, 0),
                    Text = Colorpicker.Name,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 13,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local ColorpickerPreview = CreateInstance("Frame", {
                    Parent = ColorpickerFrame,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Position = UDim2.new(1, 0, 0.5, 0),
                    Size = UDim2.new(0, 28, 0, 18),
                    BackgroundColor3 = Colorpicker.Color,
                    BorderSizePixel = 0,
                    ZIndex = 6
                })
                Round(ColorpickerPreview, 4)

                function Colorpicker:Set(color, alpha)
                    Colorpicker.Color = color
                    Colorpicker.Alpha = alpha or 0
                    ColorpickerPreview.BackgroundColor3 = color
                    Colorpicker.Callback(color, Colorpicker.Alpha)
                    if Colorpicker.Flag then OrionUI.Flags[Colorpicker.Flag] = {Color = color, Alpha = Colorpicker.Alpha} end
                end

                function Colorpicker:Get()
                    return Colorpicker.Color, Colorpicker.Alpha
                end

                return Colorpicker
            end

            -- Label
            function Section:Label(text)
                local LabelFrame = CreateInstance("TextLabel", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 20),
                    Text = text,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.SubText,
                    TextSize = 12,
                    TextWrapped = true,
                    ZIndex = 5
                })

                function LabelFrame:SetText(text)
                    LabelFrame.Text = text
                end

                return LabelFrame
            end

            -- Textbox
            function Section:Textbox(options)
                options = options or {}
                local Textbox = {
                    Name = options.Name or "Textbox",
                    Flag = options.Flag,
                    Default = options.Default or "",
                    Callback = options.Callback or function() end,
                    Value = ""
                }
                Textbox.Value = Textbox.Default

                local TextboxFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 32),
                    ZIndex = 5
                })

                local TextboxInput = CreateInstance("TextBox", {
                    Parent = TextboxFrame,
                    Position = UDim2.new(0, 0, 0, 10),
                    Size = UDim2.new(1, 0, 0, 22),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    Text = Textbox.Default,
                    PlaceholderText = Textbox.Name,
                    PlaceholderColor3 = OrionUI.Theme.SubText,
                    FontFace = OrionUI.Font,
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })
                Round(TextboxInput, 4)

                local TextboxPadding = CreateInstance("UIPadding", {
                    Parent = TextboxInput,
                    PaddingLeft = UDim.new(0, 8)
                })

                TextboxInput.FocusLost:Connect(function()
                    Textbox.Value = TextboxInput.Text
                    Textbox.Callback(Textbox.Value)
                    if Textbox.Flag then OrionUI.Flags[Textbox.Flag] = Textbox.Value end
                end)

                function Textbox:Set(value)
                    Textbox.Value = value
                    TextboxInput.Text = value
                end

                function Textbox:Get()
                    return Textbox.Value
                end

                return Textbox
            end

            return Section
        end

        return Page
    end

    -- Notification
    function OrionUI:Notification(options)
        options = options or {}
        local Notification = {
            Title = options.Title or "Notification",
            Description = options.Description or "",
            Duration = options.Duration or 3,
            Icon = options.Icon
        }

        local NotifFrame = CreateInstance("Frame", {
            Parent = ScreenGui,
            BackgroundColor3 = OrionUI.Theme.Background2,
            BorderSizePixel = 0,
            Position = UDim2.new(1, -220, 0, 20),
            Size = UDim2.new(0, 200, 0, 60),
            AutomaticSize = Enum.AutomaticSize.Y,
            ClipsDescendants = true,
            ZIndex = 100
        })
        Round(NotifFrame, 8)

        local NotifPadding = CreateInstance("UIPadding", {
            Parent = NotifFrame,
            PaddingAll = UDim.new(0, 12)
        })

        local NotifTitle = CreateInstance("TextLabel", {
            Parent = NotifFrame,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 18),
            Text = Notification.Title,
            FontFace = OrionUI.Font,
            TextColor3 = OrionUI.Theme.Text,
            TextSize = 14,
            TextXAlignment = Enum.TextXAlignment.Left,
            ZIndex = 101
        })

        local NotifDesc = CreateInstance("TextLabel", {
            Parent = NotifFrame,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 14),
            Position = UDim2.new(0, 0, 0, 20),
            Text = Notification.Description,
            FontFace = OrionUI.Font,
            TextColor3 = OrionUI.Theme.SubText,
            TextSize = 12,
            TextWrapped = true,
            ZIndex = 101
        })

        -- Animation
        NotifFrame.Position = UDim2.new(1, 0, 0, 20)
        NotifFrame:TweenPosition(UDim2.new(1, -220, 0, 20), "Out", "Quad", 0.3)

        task.delay(Notification.Duration, function()
            NotifFrame:TweenPosition(UDim2.new(1, 220, 0, 20), "In", "Quad", 0.3, true)
            task.wait(0.3)
            NotifFrame:Destroy()
        end)
    end

    -- Config
    function OrionUI:GetConfig()
        return HttpService:JSONEncode(OrionUI.Flags)
    end

    function OrionUI:LoadConfig(config)
        local success, decoded = pcall(function()
            return HttpService:JSONDecode(config)
        end)
        if success then
            for k, v in pairs(decoded) do
                OrionUI.Flags[k] = v
            end
        end
    end

    return Window
end

return OrionUI
