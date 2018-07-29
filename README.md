--[[                                                                                              
                                                                                                 
LLLLLLLLLLL                  OOOOOOOOO                    AAA               DDDDDDDDDDDDD        
L:::::::::L                OO:::::::::OO                 A:::A              D::::::::::::DDD     
L:::::::::L              OO:::::::::::::OO              A:::::A             D:::::::::::::::DD   
LL:::::::LL             O:::::::OOO:::::::O            A:::::::A            DDD:::::DDDDD:::::D  
  L:::::L               O::::::O   O::::::O           A:::::::::A             D:::::D    D:::::D 
  L:::::L               O:::::O     O:::::O          A:::::A:::::A            D:::::D     D:::::D
  L:::::L               O:::::O     O:::::O         A:::::A A:::::A           D:::::D     D:::::D
  L:::::L               O:::::O     O:::::O        A:::::A   A:::::A          D:::::D     D:::::D
  L:::::L               O:::::O     O:::::O       A:::::A     A:::::A         D:::::D     D:::::D
  L:::::L               O:::::O     O:::::O      A:::::AAAAAAAAA:::::A        D:::::D     D:::::D
  L:::::L               O:::::O     O:::::O     A:::::::::::::::::::::A       D:::::D     D:::::D
  L:::::L         LLLLLLO::::::O   O::::::O    A:::::AAAAAAAAAAAAA:::::A      D:::::D    D:::::D 
LL:::::::LLLLLLLLL:::::LO:::::::OOO:::::::O   A:::::A             A:::::A   DDD:::::DDDDD:::::D  
L::::::::::::::::::::::L OO:::::::::::::OO   A:::::A               A:::::A  D:::::::::::::::DD   
L::::::::::::::::::::::L   OO:::::::::OO    A:::::A                 A:::::A D::::::::::::DDD     
LLLLLLLLLLLLLLLLLLLLLLLL     OOOOOOOOO     AAAAAAA                   AAAAAAADDDDDDDDDDDDD        
                                                                                             
--]]                   
--v 2.6
local Heroes 						= {"Kaisa", "Galio", "Teemo", "Tristana", "Jinx", "Caitlyn", "Kayn", 
	"Syndra", "Tryndamere", "Swain", "Darius", "Jhin", "Sivir", "Nasus", "Malphite", "MasterYi", "Rengar", 
		"Pyke", "Kalista", "Aatrox", "Corki", "Leblanc", "Evelynn", "Rumble", "Irelia"}
local _atan 						= math.atan2
local _pi 							= math.pi
local _max 							= math.max
local _min 							= math.min
local _abs 							= math.abs
local _sqrt 						= math.sqrt
local _find 						= string.find
local _sub 							= string.sub
local _len 							= string.len
local _huge 						= math.huge
local _insert						= table.insert
local LocalGameLatency				= Game.Latency
local LocalGameTimer 				= Game.Timer
local charName 						= myHero.charName
local isAioLoaded 					= false
local LocalGameHeroCount 			= Game.HeroCount;
local LocalGameHero 				= Game.Hero;
local LocalGameMinionCount 			= Game.MinionCount;
local LocalGameMinion 				= Game.Minion;
local LocalGameTurretCount 			= Game.TurretCount;
local LocalGameTurret 				= Game.Turret;
local LocalGameWardCount 			= Game.WardCount;
local LocalGameWard 				= Game.Ward;
local LocalGameObjectCount 			= Game.ObjectCount;
local LocalGameObject				= Game.Object;
local LocalGameMissileCount 		= Game.MissileCount;
local LocalGameMissile				= Game.Missile;
local LocalGameParticleCount		= Game.ParticleCount;
local LocalGameParticle				= Game.Particle;
local LocalGameCampCount			= Game.CampCount;
local LocalGameCamp					= Game.Camp;
local isEvading 					= ExtLibEvade and ExtLibEvade.Evading
local _targetedMissiles = {}
local _activeSkillshots = {}

if not table.contains(Heroes, charName) then 
	print("LazyAiO - " ..charName.. " is not supported, loading other modules...")
else
	print("LazyAiO - " ..charName.. " loading...")
end

Callback.Add("Load",
function()	

	if FileExist(COMMON_PATH.. "Collision.lua") then
		require 'Collision'
	else
		print("Collision lib missing... Download and insert it in " ..COMMON_PATH.. " !")
	end	

	Menu = MenuElement({type = MENU, id = charName, name = "LazyAiO "..charName})	
	Menu:MenuElement({id = "AiO", name = "AiO Settings", type = MENU})

		Menu.AiO:MenuElement({ id = "Activator", name = "Activator", type = MENU})

			--- Items ---
			Menu.AiO.Activator:MenuElement({ id = "Items", name = "Items", type = MENU})

				--tiamat
				Menu.AiO.Activator.Items:MenuElement({ id = "tiamat", name = "Tiamat", type = MENU })
					Menu.AiO.Activator.Items.tiamat:MenuElement({ id = "useTiamat", name = "Use", value = true, tooltip = "general ON/OFF switch" })

				--ravenousHydra
				Menu.AiO.Activator.Items:MenuElement({ id = "ravenousHydra", name = "Ravenous Hydra", type = MENU })
					Menu.AiO.Activator.Items.ravenousHydra:MenuElement({ id = "useRavenousHydra", name = "Use", value = true, tooltip = "general ON/OFF switch" })

				--ravenousHydra
				Menu.AiO.Activator.Items:MenuElement({ id = "titanicHydra", name = "Titanic Hydra", type = MENU })
					Menu.AiO.Activator.Items.titanicHydra:MenuElement({ id = "useTitanicHydra", name = "Use", value = true, tooltip = "general ON/OFF switch" })

				--bilgewaterCutlass
				Menu.AiO.Activator.Items:MenuElement({ id = "bilgewaterCutlass", name = "Bilgewater Cutlass", type = MENU })
					Menu.AiO.Activator.Items.bilgewaterCutlass:MenuElement({ id = "useBilgewaterCutlass", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.bilgewaterCutlass:MenuElement({ id = "bilgewaterCutlassTHP", name = "Enemy HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "if Target HP% < x" })
					Menu.AiO.Activator.Items.bilgewaterCutlass:MenuElement({ id = "bilgewaterCutlassHP", name = "my Hero HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "My HP% < x" })

				--botrk
				Menu.AiO.Activator.Items:MenuElement({ id = "botrk", name = "BotRK", type = MENU })
					Menu.AiO.Activator.Items.botrk:MenuElement({ id = "useBotrk", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.botrk:MenuElement({ id = "botrkTHP", name = "Enemy HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "if Target HP% < x" })
					Menu.AiO.Activator.Items.botrk:MenuElement({ id = "botrkHP", name = "my Hero HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "My HP% < x" })

				--edgeOfNight
				Menu.AiO.Activator.Items:MenuElement({ id = "edgeOfNight", name = "Edge Of Night", type = MENU })
					Menu.AiO.Activator.Items.edgeOfNight:MenuElement({ id = "useEdgeOfNight", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.edgeOfNight:MenuElement({ id = "edgeOfNightRange", name = "Range", value = 800, min = 0, max = 2000, step = 100, tooltip = "Target in x Range" })
					
				--hextechGLP
				Menu.AiO.Activator.Items:MenuElement({ id = "hextechGLP", name = "Hextech GLP-800", type = MENU })
					Menu.AiO.Activator.Items.hextechGLP:MenuElement({ id = "useHextechGLP", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.hextechGLP:MenuElement({ id = "hextechGLPTHP", name = "Enemy HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "if Target HP% < x" })
					Menu.AiO.Activator.Items.hextechGLP:MenuElement({ id = "hextechGLPHP", name = "my Hero HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "My HP% < x" })
		
				--hextechGunblade
				Menu.AiO.Activator.Items:MenuElement({ id = "hextechGunblade", name = "Hextech Gunblade", type = MENU })
					Menu.AiO.Activator.Items.hextechGunblade:MenuElement({ id = "useHextechGunblade", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.hextechGunblade:MenuElement({ id = "hextechGunbladeTHP", name = "Enemy HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "if Target HP% < x" })
					Menu.AiO.Activator.Items.hextechGunblade:MenuElement({ id = "hextechGunbladeHP", name = "my Hero HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "My HP% < x" })
				
				--hextechProtobelt
				Menu.AiO.Activator.Items:MenuElement({ id = "hextechProtobelt", name = "Hextech Protobelt", type = MENU })
					Menu.AiO.Activator.Items.hextechProtobelt:MenuElement({ id = "useHextechProtobelt", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.hextechProtobelt:MenuElement({ id = "hextechProtobeltTHP", name = "Enemy HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "if Target HP% < x" })
					Menu.AiO.Activator.Items.hextechProtobelt:MenuElement({ id = "hextechProtobeltHP", name = "my Hero HP %", value = 40, min = 0, max = 100, step = 1, tooltip = "My HP% > x" })
				
				--redemption
				Menu.AiO.Activator.Items:MenuElement({ id = "redemption", name = "Redemption", type = MENU })
					Menu.AiO.Activator.Items.redemption:MenuElement({ id = "useRedemption", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.redemption:MenuElement({ id = "redemptionHP", name = "Ally HP %", value = 15, min = 0, max = 100, step = 1, tooltip = "Ally HP% < x" })

				--zhonyas
				Menu.AiO.Activator.Items:MenuElement({ id = "zhonyasHourglass", name = "Zhonyas Hourglass", type = MENU })
					Menu.AiO.Activator.Items.zhonyasHourglass:MenuElement({ id = "useZhonyasHourglass", name = "Use", value = true, tooltip = "general ON/OFF switch" })
					Menu.AiO.Activator.Items.zhonyasHourglass:MenuElement({ id = "zhonyasHourglassHP", name = "my Hero HP %", value = 10, min = 0, max = 100, step = 1, tooltip = "My HP% < x" })
					Menu.AiO.Activator.Items.zhonyasHourglass:MenuElement({ id = "zhonyasHourglassRange", name = "Range", value = 500, min = 0, max = 1000, step = 10, tooltip = "Enemy in x Range" })
				
				--QSS
				Menu.AiO.Activator.Items:MenuElement({ id = "qss", name = "Quicksilver Sash", type = MENU })
					Menu.AiO.Activator.Items.qss:MenuElement({ id = "useQss", name = "Use", value = true, tooltip = "general ON/OFF switch" })

				--mercurials
				Menu.AiO.Activator.Items:MenuElement({ id = "mercurialScimitar", name = "Mercurial Scimitar", type = MENU })
					Menu.AiO.Activator.Items.mercurialScimitar:MenuElement({ id = "useMercurialScimitar", name = "Use", value = true, tooltip = "general ON/OFF switch" })

				--mikaels
				Menu.AiO.Activator.Items:MenuElement({ id = "mikaelsCrucible", name = "Mikaels Crucible", type = MENU })
					Menu.AiO.Activator.Items.mikaelsCrucible:MenuElement({ id = "useMikaelsCrucible", name = "Use", value = true, tooltip = "general ON/OFF switch" })

			--- Summoners ---

			Menu.AiO.Activator:MenuElement({ id = "Summoners", name = "Summoners", type = MENU})
			Menu.AiO.Activator.Summoners:MenuElement({ id = "useSmite", name = "Cast red/blue smite in Combo", value = true}) -- stealth champs? TODO

			Menu.AiO:MenuElement({id = "Draw", name = "Draw Settings", type = MENU})

				Menu.AiO.Draw:MenuElement({id = "DrawQ", name = "Draw Q Range", value = false})
				Menu.AiO.Draw:MenuElement({id = "DrawW", name = "Draw W Range", value = false})
				Menu.AiO.Draw:MenuElement({id = "DrawE", name = "Draw E Range", value = false})
				Menu.AiO.Draw:MenuElement({id = "DrawR", name = "Draw R Range", value = false})

			Menu.AiO:MenuElement({id = "Pred", name = "Prediction Settings", type = MENU})
				Menu.AiO.Pred:MenuElement({id = "pred", name = "Pred to use",  drop = {"Noddy's", "HPred"}, value = 2})

				Menu.AiO.Pred:MenuElement({id = "HPred", name = "HPred Settings", type = MENU})
				Menu.AiO.Pred.HPred:MenuElement({id = "reactionTime", name = "Reaction Time", value = .5, min = .1, max = 1, step = .1})
				Menu.AiO.Pred.HPred:MenuElement({id = "accuracy", name = "HitChance", value = 3, min = 1, max = 5, step = 1})

				Menu.AiO.Pred:MenuElement({id = "NPred", name = "Noddy's Pred Settings", type = MENU})
				Menu.AiO.Pred.NPred:MenuElement({id = "soonTM", name = "Nothing here :P", value = false})
	HPred()
	AioLoad()
end)

function AioLoad()
	if not isAioLoaded then
		LoadAio()
		return
	end
end

function LoadAio()

	isAioLoaded = true
	if table.contains(Heroes, charName) then
		_G[charName]()
	end

	Callback.Add("Tick", function() Utils:OnTick() end)
	if _G.SDK then
        _G.SDK.Orbwalker:OnPostAttack(function() Utils:OnPostAttack() end)
    elseif _G.EOW then
        EOW:AddCallback(EOW.AfterAttack, function() Utils:OnPostAttack() end)
    elseif _G.GOS then
        GOS:OnAttackComplete(function() Utils:OnPostAttack() end)
    end
	Callback.Add("Tick", function() HPred:Tick() end)
	Callback.Add("Draw", function() AioDraw() end)
	LoadSummonersAiO()
end

function LoadSummonersAiO()

	Utils.Smite = Utils:GetSmite(SUMMONER_1) or 0
	if Utils.Smite == 0 then
		Utils.Smite = Utils:GetSmite(SUMMONER_2);		
	end
	Utils.lastSmite = GetTickCount()

	Utils.Ignite = Utils:GetIgnite(SUMMONER_1) or 0
	if Utils.Ignite == 0 then
		Utils.Ignite = Utils:GetIgnite(SUMMONER_2);		
	end
	Utils.Ignite = GetTickCount()
end

function AioDraw()

	if Q and Q.range and Menu.AiO.Draw.DrawQ:Value() then
		if Utils:IsReady(_Q) then
			Draw.Circle(myHero.pos, Q.range, 1, Draw.Color(255, 0, 255, 0))
		else
			Draw.Circle(myHero.pos, Q.range, 1, Draw.Color(255, 255, 0, 0))
		end
	end	
	if W and W.range and Menu.AiO.Draw.DrawW:Value() then
		if Utils:IsReady(_W) then
			Draw.Circle(myHero.pos, W.range, 1, Draw.Color(255, 0, 255, 0))
		else
			Draw.Circle(myHero.pos, W.range, 1, Draw.Color(255, 255, 0, 0))
		end
	end	
	if E and E.range and Menu.AiO.Draw.DrawE:Value() then
		if Utils:IsReady(_E) then
			Draw.Circle(myHero.pos, E.range, 1, Draw.Color(255, 0, 255, 0))
		else
			Draw.Circle(myHero.pos, E.range, 1, Draw.Color(255, 255, 0, 0))
		end
	end		
	if R and R.range and Menu.AiO.Draw.DrawR:Value() then
		if Utils:IsReady(_R) then
			Draw.Circle(myHero.pos, R.range, 1, Draw.Color(255, 0, 255, 0))
		else
			Draw.Circle(myHero.pos, R.range, 1, Draw.Color(255, 255, 0, 0))
		end
	end
end

--[[                                                                                               
                                                                                                 
UUUUUUUU     UUUUUUUUTTTTTTTTTTTTTTTTTTTTTTTIIIIIIIIIILLLLLLLLLLL                SSSSSSSSSSSSSSS 
U::::::U     U::::::UT:::::::::::::::::::::TI::::::::IL:::::::::L              SS:::::::::::::::S
U::::::U     U::::::UT:::::::::::::::::::::TI::::::::IL:::::::::L             S:::::SSSSSS::::::S
UU:::::U     U:::::UUT:::::TT:::::::TT:::::TII::::::IILL:::::::LL             S:::::S     SSSSSSS
 U:::::U     U:::::U TTTTTT  T:::::T  TTTTTT  I::::I    L:::::L               S:::::S            
 U:::::D     D:::::U         T:::::T          I::::I    L:::::L               S:::::S            
 U:::::D     D:::::U         T:::::T          I::::I    L:::::L                S::::SSSS         
 U:::::D     D:::::U         T:::::T          I::::I    L:::::L                 SS::::::SSSSS    
 U:::::D     D:::::U         T:::::T          I::::I    L:::::L                   SSS::::::::SS  
 U:::::D     D:::::U         T:::::T          I::::I    L:::::L                      SSSSSS::::S 
 U:::::D     D:::::U         T:::::T          I::::I    L:::::L                           S:::::S
 U::::::U   U::::::U         T:::::T          I::::I    L:::::L         LLLLLL            S:::::S
 U:::::::UUU:::::::U       TT:::::::TT      II::::::IILL:::::::LLLLLLLLL:::::LSSSSSSS     S:::::S
  UU:::::::::::::UU        T:::::::::T      I::::::::IL::::::::::::::::::::::LS::::::SSSSSS:::::S
    UU:::::::::UU          T:::::::::T      I::::::::IL::::::::::::::::::::::LS:::::::::::::::SS 
      UUUUUUUUU            TTTTTTTTTTT      IIIIIIIIIILLLLLLLLLLLLLLLLLLLLLLLL SSSSSSSSSSSSSSS   
                                                                                              
--]]

class "Utils"

function Utils:IsReady(spell)
	return Game.CanUseSpell(spell) == 0
end

function Utils:IsValid(unit, pos, range)
	return self:GetDistance(unit.pos, pos) <= range and unit.health > 0 and unit.isTargetable and unit.visible
end

function Utils:GetDistanceSqr(p1, p2)
    local dx = p1.x - p2.x
    local dz = p1.z - p2.z
    return (dx * dx + dz * dz)
end

function Utils:GetDistance(p1, p2)
    return _sqrt(self:GetDistanceSqr(p1, p2))
end

function Utils:GetDistance2D(p1,p2)
    return _sqrt((p2.x - p1.x)*(p2.x - p1.x) + (p2.y - p1.y)*(p2.y - p1.y))
end

function Utils:GetHpPercent(unit)
    return unit.health / unit.maxHealth * 100
end

function Utils:GetManaPercent(unit)
	return unit.mana / unit.maxMana * 100
end

function Utils:GetTarget(range)

	if _G.EOWLoaded then
		if myHero.ap > myHero.totalDamage then
			return EOW:GetTarget(range, EOW.ap_dec, myHero.pos)
		else
			return EOW:GetTarget(range, EOW.ad_dec, myHero.pos)
		end
	elseif _G.SDK and _G.SDK.TargetSelector then
		if myHero.ap > myHero.totalDamage then
			return _G.SDK.TargetSelector:GetTarget(range, _G.SDK.DAMAGE_TYPE_MAGICAL)
		else
			return _G.SDK.TargetSelector:GetTarget(range, _G.SDK.DAMAGE_TYPE_PHYSICAL)
		end
	elseif _G.GOS then
		if myHero.ap > myHero.totalDamage then
			return GOS:GetTarget(range, "AP")
		else
			return GOS:GetTarget(range, "AD")
		end
	end
end

function Utils:HasBuff(unit, buffName)

	for i = 1, unit.buffCount do
		local buff = unit:GetBuff(i)
		if buff and buff.count > 0 and buff.name:lower() == buffName:lower() then 
			return true
		end
	end
	return false
end

function Utils:IsParticleInRange(particleName, pos, range)

	for i = 1, LocalGameParticleCount() do
    	local particle = LocalGameParticle(i)
		if particle and not particle.dead then
			if particle.name:lower() == particleName:lower() then
				if self:GetDistance(particle.pos, pos) <= range then 
					return true
				end
			end
		end
	end
	return false
end

local _AllyHeroes
function Utils:GetAllyHeroes()
  	if #_AllyHeroes > 0 then return _AllyHeroes end
  	_AllyHeroes = {}
  	for i = 1, LocalGameHeroCount() do
    	local unit = LocalGameHero(i)
    	if unit and unit.isAlly then
      		_insert(_AllyHeroes, unit)
    	end
	end
	return _AllyHeroes
end

local _EnemyHeroes
function Utils:GetEnemyHeroes()
  	if #_EnemyHeroes > 0 then return _EnemyHeroes end
  	_EnemyHeroes = {}
  	for i = 1, LocalGameHeroCount() do
    	local unit = LocalGameHero(i)
    	if unit and unit.isEnemy then
      		_insert(_EnemyHeroes, unit)
    	end
	end
	return _EnemyHeroes
end

function Utils:GetEnemyHeroesInRange(pos, range)
	local _EnemyHeroes = {}
  	for i = 1, LocalGameHeroCount() do
    	local unit = LocalGameHero(i)
    	if unit and unit.isEnemy and self:IsValid(unit, pos, range) then
	  		_insert(_EnemyHeroes, unit)
  		end
  	end
  	return _EnemyHeroes
end

function Utils:GetAllyHeroesInRange(pos, range)
	local _AllyHeroes = {}	
  	for i = 1, LocalGameHeroCount() do
    	local unit = LocalGameHero(i)
    	if unit and unit.isAlly and self:IsValid(unit, pos, range) then
	  		_insert(_AllyHeroes, unit)
  		end
  	end
  	return _AllyHeroes
end

function Utils:GetEnemyMinionsInRange(pos, range)

	local _EnemyMinions = {}
  	for i = 1, LocalGameMinionCount() do
    	local unit = LocalGameMinion(i)
    	if unit and unit.isEnemy and self:IsValid(unit, pos, range) then
	  		_insert(_EnemyMinions, unit)
  		end
  	end
  	return _EnemyMinions
end

function Utils:GetAllyMinionsInRange(pos, range)

	local _AllyMinions = {}	
  	for i = 1, LocalGameMinionCount() do
    	local unit = LocalGameMinion(i)
    	if unit and unit.isAlly and self:IsValid(unit, pos, range) then
	  		_insert(_AllyMinions, unit)
  		end
  	end
  	return _AllyMinions
end

function Utils:IsUnderEnemyTurret(pos)

	for i = 1, LocalGameTurretCount() do
		local turret = LocalGameTurret(i);
		if turret then
			if turret.valid and turret.health > 0 and turret.isEnemy then
				local turretPos = turret.pos
				if Utils:GetDistance(pos, turretPos) <= 900 then 
					return true
				end
			end
		end
	end
	return false
end

function Utils:IsUnderAllyTurret(pos)

	for i = 1, LocalGameTurretCount() do
		local turret = LocalGameTurret(i);
		if turret then
			if turret.valid and turret.health > 0 and turret.isAlly then
				local turretPos = turret.pos
				if Utils:GetDistance(pos, turretPos) <= 900 then 
					return true
				end
			end
		end
	end
	return false
end

local castSpell = {state = 0, tick = GetTickCount(), casting = GetTickCount() - 1000, mouse = mousePos}
function Utils:CastSpell(spell,pos,range,delay)

	local range = range or _huge
	local delay = delay or 250
	local ticker = GetTickCount()

	if castSpell.state == 0 and self:GetDistance(myHero.pos, pos) < range and ticker - castSpell.casting > delay + LocalGameLatency() then
		castSpell.state = 1
		castSpell.mouse = mousePos
		castSpell.tick = ticker
	end
	if castSpell.state == 1 then
		if ticker - castSpell.tick < LocalGameLatency() then
			Control.SetCursorPos(pos)
			Control.KeyDown(spell)
			Control.KeyUp(spell)
			castSpell.casting = ticker + delay
			DelayAction(function()
				if castSpell.state == 1 then
					Control.SetCursorPos(castSpell.mouse)
					castSpell.state = 0
				end
			end,LocalGameLatency()/1000)
		end
		if ticker - castSpell.casting > LocalGameLatency() then
			Control.SetCursorPos(castSpell.mouse)
			castSpell.state = 0
		end
	end
end

function Utils:CastSpellMM(spell, pos, range, delay)

	local range = range or _huge
	local delay = delay or 250
	local ticker = GetTickCount()

	if castSpell.state == 0 and self:GetDistance(myHero.pos, pos) < range and ticker - castSpell.casting > delay + LocalGameLatency() then
		castSpell.state = 1
		castSpell.mouse = mousePos
		castSpell.tick = ticker
	end
	if castSpell.state == 1 then
		if ticker - castSpell.tick < LocalGameLatency() then
			local castPosMM = pos:ToMM()
			Control.SetCursorPos(castPosMM.x,castPosMM.y)
			Control.KeyDown(spell)
			Control.KeyUp(spell)
			castSpell.casting = ticker + delay
			DelayAction(function()
				if castSpell.state == 1 then
					Control.SetCursorPos(castSpell.mouse)
					castSpell.state = 0
				end
			end,LocalGameLatency()/1000)
		end
		if ticker - castSpell.casting > LocalGameLatency() then
			Control.SetCursorPos(castSpell.mouse)
			castSpell.state = 0
		end
	end
end

function Utils:CastSpellCharged(spell, pos, range, delay)

	local range = range or _huge
	local delay = delay or 250
	local ticker = GetTickCount()

	if castSpell.state == 0 and self:GetDistance(myHero.pos, pos) < range and ticker - castSpell.casting > delay + LocalGameLatency() then
		castSpell.state = 1
		castSpell.mouse = mousePos
		castSpell.tick = ticker
	end
	if castSpell.state == 1 then
		if ticker - castSpell.tick < LocalGameLatency() then
			Control.SetCursorPos(pos)
			Control.KeyUp(spell)
			castSpell.casting = ticker + delay
			DelayAction(function()
				if castSpell.state == 1 then
					Control.SetCursorPos(castSpell.mouse)
					castSpell.state = 0
				end
			end,LocalGameLatency()/1000)
		end
		if ticker - castSpell.casting > LocalGameLatency() then
			Control.SetCursorPos(castSpell.mouse)
			castSpell.state = 0
		end
	end
end



 -- Thnx Noddy :)

local SmiteNames = {'SummonerSmite','S5_SummonerSmiteDuel','S5_SummonerSmitePlayerGanker','S5_SummonerSmiteQuick','ItemSmiteAoE'};
local SmiteDamage = {390 , 410 , 430 , 450 , 480 , 510 , 540 , 570 , 600 , 640 , 680 , 720 , 760 , 800 , 850 , 900 , 950 , 1000};

function Utils:GetSmite(smiteSlot)
	local smite = 0;
	local spellName = myHero:GetSpellData(smiteSlot).name;
	for i = 1, 5 do
		if spellName == SmiteNames[i] then
			smite = smiteSlot
		end
	end
	return smite;
end

function Utils:GetIgnite(igniteSlot)
	local ignite = 0;
	local spellName = myHero:GetSpellData(igniteSlot).name;
	if spellName:lower() == "summonerignite" then
		ignite = smiteSlot
	end
	return ignite;
end

function Utils:CastSmite()

	local sData = myHero:GetSpellData(Utils.Smite);
	if sData.name == SmiteNames[3] or sData.name == SmiteNames[2] then
		if sData.level > 0 then
			if (sData.ammo > 0 and sData.ammoCurrentCd == 0) then
				for i = 1, LocalGameHeroCount() do
					local hero = LocalGameHero(i);
					if hero and hero.isEnemy and self:IsValid(hero, myHero.pos, 500 + (myHero.boundingRadius + hero.boundingRadius)) then
						if Utils.Smite == SUMMONER_1 then
							Control.CastSpell(HK_SUMMONER_1, hero)
							self.lastSmite = GetTickCount()
						else
							Control.CastSpell(HK_SUMMONER_2, hero)
							self.lastSmite = GetTickCount()
						end
					end
				end
			end
		end
	end
end

function Utils:CalculatePhysicalDamage(target, damage)

	if target and damage then
		local targetArmor = target.armor * myHero.armorPenPercent - myHero.armorPen
		local damageReduction = 100 / ( 100 + targetArmor)
		if targetArmor < 0 then
			damageReduction = 2 - (100 / (100 - targetArmor))
		end		
		damage = damage * damageReduction	
		return damage
	end
	return 0
end

function Utils:CalculateMagicalDamage(target, damage)
	
	if target and damage then	
		local targetMR = target.magicResist * myHero.magicPenPercent - myHero.magicPen
		local damageReduction = 100 / ( 100 + targetMR)
		if targetMR < 0 then
			damageReduction = 2 - (100 / (100 - targetMR))
		end		
		damage = damage * damageReduction
		return damage
	end
	return 0
end

function Utils:IsImmobile(unit)

	for i = 1, unit.buffCount do
		local buff = unit:GetBuff(i)
		if buff and (buff.type == 5 or buff.type == 11 or buff.type == 22 or buff.type == 29) and buff.count > 0 then
			return true
		end
	end
	return false	
end

function Utils:OnVision(unit)

	local _OnVision = {}
	if _OnVision[unit.networkID] == nil then _OnVision[unit.networkID] = {state = unit.visible , tick = GetTickCount(), pos = unit.pos} end
	if _OnVision[unit.networkID].state == true and not unit.visible then _OnVision[unit.networkID].state = false _OnVision[unit.networkID].tick = GetTickCount() end
	if _OnVision[unit.networkID].state == false and unit.visible then _OnVision[unit.networkID].state = true _OnVision[unit.networkID].tick = GetTickCount() end
	return _OnVision[unit.networkID]
end

function Utils:OnVisionF()

	local visionTick = GetTickCount()
	if GetTickCount() - visionTick > 100 then
		for k, v in pairs(self:GetEnemyHeroes()) do
			if v then
				self:OnVision(v)
			end
		end
	end
end

function Utils:OnWaypoint(unit)

	local _OnWaypoint = {}
	if _OnWaypoint[unit.networkID] == nil then _OnWaypoint[unit.networkID] = {pos = unit.posTo , speed = unit.ms, time = LocalGameTimer()} end
	if _OnWaypoint[unit.networkID].pos ~= unit.posTo then 
		_OnWaypoint[unit.networkID] = {startPos = unit.pos, pos = unit.posTo , speed = unit.ms, time = LocalGameTimer()}
			DelayAction(function()
				local time = (LocalGameTimer() - _OnWaypoint[unit.networkID].time)
				local speed = self:GetDistance2D(_OnWaypoint[unit.networkID].startPos,unit.pos)/(LocalGameTimer() - _OnWaypoint[unit.networkID].time)
				if speed > 1250 and time > 0 and unit.posTo == _OnWaypoint[unit.networkID].pos and self:GetDistance(unit.pos,_OnWaypoint[unit.networkID].pos) > 200 then
					_OnWaypoint[unit.networkID].speed = self:GetDistance2D(_OnWaypoint[unit.networkID].startPos,unit.pos)/(LocalGameTimer() - _OnWaypoint[unit.networkID].time)
				end
			end,0.05)
	end
	return _OnWaypoint[unit.networkID]
end -- not sure what or why removed stuff, who cares, works.

function Utils:GetPred(unit,speed,delay,sourcePos)

	local speed = speed or _huge
	local delay = delay or 0.25
	local sourcePos = sourcePos or myHero.pos
	local unitSpeed = unit.ms

	if self:OnWaypoint(unit).speed > unitSpeed then unitSpeed = self:OnWaypoint(unit).speed end

	if unitSpeed > unit.ms then
		local predPos = unit.pos + Vector(self:OnWaypoint(unit).startPos,unit.posTo):Normalized() * (unitSpeed * (delay + (self:GetDistance(sourcePos,unit.pos)/speed)))
		if self:GetDistance(unit.pos,predPos) > self:GetDistance(unit.pos,unit.posTo) then predPos = unit.posTo end
		return predPos
	elseif self:IsImmobile(unit) then
		return unit.pos
	else
		return unit:GetPrediction(speed,delay)
	end
end

local items = { [ITEM_1] = HK_ITEM_1, [ITEM_2] = HK_ITEM_2, [ITEM_3] = HK_ITEM_3, [ITEM_4] = HK_ITEM_4, [ITEM_5] = HK_ITEM_5, [ITEM_6] = HK_ITEM_6 }
function Utils:GetInventoryItem(itemID)

	for k, v in pairs({ ITEM_1, ITEM_2, ITEM_3, ITEM_4, ITEM_5, ITEM_6 }) do
		if v then
			if myHero:GetItemData(v).itemID == itemID and myHero:GetSpellData(v).currentCd == 0 then return v end
		end
	end
	return nil
end

function Utils:OnPostAttack()
	
	local titanicHydra = self:GetInventoryItem(3748)
	local ravenousHydra = self:GetInventoryItem(3074)
	local tiamat = self:GetInventoryItem(3077)

	if Orb.combo or Orb.harass then
		for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, 550) do
			local target = Utils:GetEnemyHeroesInRange(myHero.pos, 550)[i]
			if target then
				local range = myHero.range + myHero.boundingRadius

				--ravenousHydra	
				if ravenousHydra and Menu.AiO.Activator.Items.ravenousHydra.useRavenousHydra:Value() and self:IsValid(target, myHero.pos, 550) then
					Control.CastSpell(items[ravenousHydra])
				end

				--tiamat
				if tiamat and Menu.AiO.Activator.Items.tiamat.useTiamat:Value() and self:IsValid(target, myHero.pos, 550) then
					Control.CastSpell(items[tiamat])
				end

				--titanicHydra
				if titanicHydra and Menu.AiO.Activator.Items.titanicHydra.useTitanicHydra:Value() and self:IsValid(target, myHero.pos, range) then
					Control.CastSpell(items[titanicHydra])
				end
			end
		end
	end
end

function Utils:CastItems()

	local target = nil
	local ally = nil
	local attackTarget = myHero.attackData.target
	local redemption = self:GetInventoryItem(3107)
	local bilgewaterCutlass = self:GetInventoryItem(3144)
	local botrk = self:GetInventoryItem(3153)
	--local ironSolari = self:GetInventoryItem(3383)
	local edgeOfNight = self:GetInventoryItem(3814)
	local hextechGLP = self:GetInventoryItem(3030)
	local hextechGunblade = self:GetInventoryItem(3146)
	local hextechProtobelt = self:GetInventoryItem(3152)
	local mercurialScimitar = self:GetInventoryItem(3139)
	local mikaelsCrucible = self:GetInventoryItem(3222)
	--local youmuusGhostblade = self:GetInventoryItem(3142)
	local zhonyasHourglass = self:GetInventoryItem(3157)
	local qss = self:GetInventoryItem(3140)

	for _, v in pairs(self:GetEnemyHeroesInRange(myHero.pos, 5500)) do
		local current = v
		if current and self:IsValid(current, myHero.pos, 5500) then
			target = current
		end
	end

	for _, v in pairs(self:GetAllyHeroesInRange(myHero.pos, 5500)) do
		local current = v
		if current and self:IsValid(current, myHero.pos, 5500) then
			ally = current
		end
	end

	--redemption
		if redemption and Menu.AiO.Activator.Items.redemption.useRedemption:Value() then
			if ally and self:GetHpPercent(ally) <= Menu.AiO.Activator.Items.redemption.redemptionHP:Value() then
				if ally.pos:To2D().onScreen then
					self:CastSpell(items[redemption], ally.pos, 5500)
				else
					self:CastSpellMM(items[redemption], ally.pos, 5500)
				end
			end
		end

	if myHero.alive then
		--QSS
		if qss and Menu.AiO.Activator.Items.qss.useQss:Value() and self:IsImmobile(myHero) and #Utils:GetEnemyHeroesInRange(myHero.pos, 2000) > 0 then
			Control.CastSpell(items[qss])
		end

		--mercurialScimitar
		if mercurialScimitar and Menu.AiO.Activator.Items.mercurialScimitar.useMercurialScimitar:Value() and self:IsImmobile(myHero) and 
			#Utils:GetEnemyHeroesInRange(myHero.pos, 2000) > 0 then
			Control.CastSpell(items[mercurialScimitar])
		end

		--mikaels
		if mikaelsCrucible and ally and ally.networkID ~= myHero.networkID and Menu.AiO.Activator.Items.mikaelsCrucible.useMikaelsCrucible:Value() and self:IsImmobile(ally) and 
			#Utils:GetEnemyHeroesInRange(ally.pos, 2000) > 0 and
			self:GetDistance(myHero.pos, ally.pos) <= 750 then
			self:CastSpell(items[mikaelsCrucible], ally.pos, 750)
		end
	end

	if target and myHero.alive then 

	--bilgewaterCutlass
		if bilgewaterCutlass and Menu.AiO.Activator.Items.bilgewaterCutlass.useBilgewaterCutlass:Value() and self:IsValid(target, myHero.pos, 550) then
			if self:GetHpPercent(target) <= Menu.AiO.Activator.Items.bilgewaterCutlass.bilgewaterCutlassTHP:Value() or
				self:GetHpPercent(myHero) <= Menu.AiO.Activator.Items.bilgewaterCutlass.bilgewaterCutlassHP:Value() then
				self:CastSpell(items[bilgewaterCutlass], target.pos, 550)
			end
		end

	--botrk
		if botrk and Menu.AiO.Activator.Items.botrk.useBotrk:Value() and self:IsValid(target, myHero.pos, 550) then
			if self:GetHpPercent(target) <= Menu.AiO.Activator.Items.botrk.botrkTHP:Value() or
				self:GetHpPercent(myHero) <= Menu.AiO.Activator.Items.botrk.botrkHP:Value() then
				self:CastSpell(items[botrk], target.pos, 550)
			end
		end

	--edgeOfNight
		if edgeOfNight and Menu.AiO.Activator.Items.edgeOfNight.useEdgeOfNight:Value() and
			self:IsValid(target, myHero.pos, Menu.AiO.Activator.Items.edgeOfNight.edgeOfNightRange:Value()) then
			Control.CastSpell(items[edgeOfNight])
		end

	--hextechGLP
		if hextechGLP and Menu.AiO.Activator.Items.hextechGLP.useHextechGLP:Value() and self:IsValid(target, myHero.pos, 550) then
			if self:GetHpPercent(target) <= Menu.AiO.Activator.Items.hextechGLP.hextechGLPTHP:Value() or
				self:GetHpPercent(myHero) <= Menu.AiO.Activator.Items.hextechGLP.hextechGLPHP:Value() then
				self:CastSpell(items[hextechGLP], target.pos, 550)
			end
		end

	--hextechGunblade
		if hextechGunblade and Menu.AiO.Activator.Items.hextechGunblade.useHextechGunblade:Value() and self:IsValid(target, myHero.pos, 700) then
			if self:GetHpPercent(target) <= Menu.AiO.Activator.Items.hextechGunblade.hextechGunbladeTHP:Value() or
				self:GetHpPercent(myHero) <= Menu.AiO.Activator.Items.hextechGunblade.hextechGunbladeHP:Value() then
				self:CastSpell(items[hextechGunblade], target.pos, 700)
			end
		end

	--hextechProtobelt
		if hextechProtobelt and Menu.AiO.Activator.Items.hextechProtobelt.useHextechProtobelt:Value() and self:IsValid(target, myHero.pos, 550) then
			if self:GetHpPercent(target) <= Menu.AiO.Activator.Items.hextechProtobelt.hextechProtobeltTHP:Value() and 
				self:GetHpPercent(myHero) >= Menu.AiO.Activator.Items.hextechProtobelt.hextechProtobeltHP:Value() then
				self:CastSpell(items[hextechProtobelt], target.pos, 550)
			end
		end

	--zhonyas
		if zhonyas and Menu.AiO.Activator.Items.zhonyas.useZhonyas:Value() and
			self:GetDistance(myHero.pos, target.pos) <= Menu.AiO.Activator.Items.zhonyas.zhonyasRange:Value() then
			if self:GetHpPercent(myHero) <= Menu.AiO.Activator.Items.zhonyas.zhonyasHP:Value() then
				Control.CastSpell(items[zhonyasHourglass])
			end
		end
	end
end

function Utils:OnTick()

	Orb:GetOrbMode()
	self:OnVisionF()
	self:CastItems()

	if Orb.combo then
		if Menu.AiO.Activator.Summoners.useSmite:Value() and Orb.combo then
			Utils:CastSmite()
		end
	end
end

--[[
                                                                                                                
     OOOOOOOOO     RRRRRRRRRRRRRRRRR   BBBBBBBBBBBBBBBBB   
   OO:::::::::OO   R::::::::::::::::R  B::::::::::::::::B  
 OO:::::::::::::OO R::::::RRRRRR:::::R B::::::BBBBBB:::::B 
O:::::::OOO:::::::ORR:::::R     R:::::RBB:::::B     B:::::B
O::::::O   O::::::O  R::::R     R:::::R  B::::B     B:::::B
O:::::O     O:::::O  R::::R     R:::::R  B::::B     B:::::B
O:::::O     O:::::O  R::::RRRRRR:::::R   B::::BBBBBB:::::B 
O:::::O     O:::::O  R:::::::::::::RR    B:::::::::::::BB  
O:::::O     O:::::O  R::::RRRRRR:::::R   B::::BBBBBB:::::B 
O:::::O     O:::::O  R::::R     R:::::R  B::::B     B:::::B
O:::::O     O:::::O  R::::R     R:::::R  B::::B     B:::::B
O::::::O   O::::::O  R::::R     R:::::R  B::::B     B:::::B
O:::::::OOO:::::::ORR:::::R     R:::::RBB:::::BBBBBB::::::B
 OO:::::::::::::OO R::::::R     R:::::RB:::::::::::::::::B 
   OO:::::::::OO   R::::::R     R:::::RB::::::::::::::::B  
     OOOOOOOOO     RRRRRRRR     RRRRRRRBBBBBBBBBBBBBBBBB   
                                                                                                                                                                              
--]]

class "Orb"

function Orb:DisableMovement(bool)

	if _G.SDK then
		_G.SDK.Orbwalker:SetMovement(not bool)
	elseif _G.EOWLoaded then
		EOW:SetMovements(not bool)
	elseif _G.GOS then
		GOS.BlockMovement = bool
	end
end

function Orb:DisableAttacks(bool)

	if _G.SDK then
		_G.SDK.Orbwalker:SetAttack(not bool)
	elseif _G.EOWLoaded then
		EOW:SetAttacks(not bool)
	elseif _G.GOS then
		GOS.BlockAttack = bool
	end
end

function Orb:GetOrbMode()

	self.combo, self.harass, self.lastHit, self.laneClear, self.jungleClear, self.canMove, self.canAttack = nil,nil,nil,nil,nil,nil,nil
		
	if _G.EOWLoaded then

		local mode = EOW:Mode()

		self.combo = mode == 1
		self.harass = mode == 2
	    self.lastHit = mode == 3
	    self.laneClear = mode == 4
	    self.jungleClear = mode == 4

		self.canmove = EOW:CanMove()
		self.canattack = EOW:CanAttack()
	elseif _G.SDK then

		self.combo = _G.SDK.Orbwalker.Modes[_G.SDK.ORBWALKER_MODE_COMBO]
		self.harass = _G.SDK.Orbwalker.Modes[_G.SDK.ORBWALKER_MODE_HARASS]
	   	self.lastHit = _G.SDK.Orbwalker.Modes[_G.SDK.ORBWALKER_MODE_LASTHIT]
	   	self.laneClear = _G.SDK.Orbwalker.Modes[_G.SDK.ORBWALKER_MODE_LANECLEAR]
	   	self.jungleClear = _G.SDK.Orbwalker.Modes[_G.SDK.ORBWALKER_MODE_JUNGLECLEAR]

		self.canmove = _G.SDK.Orbwalker:CanMove(myHero)
		self.canattack = _G.SDK.Orbwalker:CanAttack(myHero)
	elseif _G.GOS then

		local mode = GOS:GetMode()

		self.combo = mode == "Combo"
		self.harass = mode == "Harass"
	    self.lastHit = mode == "Lasthit"
	    self.laneClear = mode == "Clear"
	    self.jungleClear = mode == "Clear"

		self.canMove = GOS:CanMove()
		self.canAttack = GOS:CanAttack()	
	end
end

--[[                                                                                                                 
                                                                                                                   
KKKKKKKKK    KKKKKKK               AAA               IIIIIIIIII   SSSSSSSSSSSSSSS                AAA               
K:::::::K    K:::::K              A:::A              I::::::::I SS:::::::::::::::S              A:::A              
K:::::::K    K:::::K             A:::::A             I::::::::IS:::::SSSSSS::::::S             A:::::A             
K:::::::K   K::::::K            A:::::::A            II::::::IIS:::::S     SSSSSSS            A:::::::A            
KK::::::K  K:::::KKK           A:::::::::A             I::::I  S:::::S                       A:::::::::A           
  K:::::K K:::::K             A:::::A:::::A            I::::I  S:::::S                      A:::::A:::::A          
  K::::::K:::::K             A:::::A A:::::A           I::::I   S::::SSSS                  A:::::A A:::::A         
  K:::::::::::K             A:::::A   A:::::A          I::::I    SS::::::SSSSS            A:::::A   A:::::A        
  K:::::::::::K            A:::::A     A:::::A         I::::I      SSS::::::::SS         A:::::A     A:::::A       
  K::::::K:::::K          A:::::AAAAAAAAA:::::A        I::::I         SSSSSS::::S       A:::::AAAAAAAAA:::::A      
  K:::::K K:::::K        A:::::::::::::::::::::A       I::::I              S:::::S     A:::::::::::::::::::::A     
KK::::::K  K:::::KKK    A:::::AAAAAAAAAAAAA:::::A      I::::I              S:::::S    A:::::AAAAAAAAAAAAA:::::A    
K:::::::K   K::::::K   A:::::A             A:::::A   II::::::IISSSSSSS     S:::::S   A:::::A             A:::::A   
K:::::::K    K:::::K  A:::::A               A:::::A  I::::::::IS::::::SSSSSS:::::S  A:::::A               A:::::A  
K:::::::K    K:::::K A:::::A                 A:::::A I::::::::IS:::::::::::::::SS  A:::::A                 A:::::A 
KKKKKKKKK    KKKKKKKAAAAAAA                   AAAAAAAIIIIIIIIII SSSSSSSSSSSSSSS   AAAAAAA                   AAAAAAA                                                                                                                
                                                                                                                   
--]]                                                                                 
                                                                                                                                                                                  
class "Kaisa"

function Kaisa:__init()

	if charName ~= "Kaisa" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Kaisa:LoadSpells()

	Q = { range = 650 }
	W = { range = 3000,	width = myHero:GetSpellData(_W).width,	speed = myHero:GetSpellData(_W).speed,	delay = 0.25, 
		collision = Collision:SetSpell(3000, myHero:GetSpellData(_W).speed, 0.25, myHero:GetSpellData(_W).width, true) }
	E = { range = myHero:GetSpellData(_E).range }
	R = { range = myHero:GetSpellData(_R).range }
end

function Kaisa:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
			Menu.Combo:MenuElement({id = "rSettings", name = "R settings", type = MENU})
			Menu.Combo.rSettings:MenuElement({id = "useR", name = "Use R", value = true})
			Menu.Combo.rSettings:MenuElement({id = "def", name = "Defensive usage", type = MENU})
				Menu.Combo.rSettings.def:MenuElement({id = "useRdefensive", name = "Use R defensive", value = true})
				Menu.Combo.rSettings.def:MenuElement({id = "useRdefensiveHp", name = "HP% for defensive usage", value = 20, min = 0, max = 100, step = 1})
				Menu.Combo.rSettings.def:MenuElement({id = "useRdefensiveRange", name = "if enemy in X range", value = 600, min = 0, max = 3000, step = 50})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})

	--- Misc ---
		Menu.Misc:MenuElement({id = "drawLine", name = "Draw line if can use R", value = true })
		Menu.Misc:MenuElement({id = "lineWidth", name = "Line width", value = 2, min = 1, max = 10, step = 1 })

end

function Kaisa:OnTick()

	local _hasEbuff = Utils:HasBuff(myHero, "kaisae")
	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)

	if _hasEbuff then
		Orb:DisableAttacks(true)
	else
		Orb:DisableAttacks(false)
	end


	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Kaisa:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		Control.CastSpell(HK_Q)
	end
end

function Kaisa:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, W.speed, W.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					local col, units = W.collision:__GetCollision(myHero.pos, nPred, 5)
					if not col then
						if  Utils:GetDistance(nPred, myHero.pos) < W.range then
							if nPred:To2D().onScreen then
								Utils:CastSpell(HK_W, nPred, W.range)
							else
								Utils:CastSpellMM(HK_W, nPred, W.range)
							end
						end		
					end
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					local col, units = W.collision:__GetCollision(myHero.pos, aimPos, 5)
					if not col then
						if Utils:GetDistance(aimPos, myHero.pos) < W.range then
							if aimPos:To2D().onScreen then
								Utils:CastSpell(HK_W, aimPos, W.range)
							else
								Utils:CastSpellMM(HK_W, aimPos, W.range)
							end
						end
					end
				end
				if hitChance and aimPos2 then
					local col, units = W.collision:__GetCollision(myHero.pos, aimPos2, 5)
					if not col then
						if hitChance >= minAccuracy or (hitChance >= 1 and self:GetWdmg(unit) > unit.health) then
							if Utils:GetDistance(aimPos2, myHero.pos) < W.range then
								if aimPos2:To2D().onScreen then
									Utils:CastSpell(HK_W, aimPos2, W.range)
								else
									Utils:CastSpellMM(HK_W, aimPos2, W.range)
								end
							end
						end
					end
				end
			end
		end
	end
end

function Kaisa:CastDefensiveR()
	
	for _, v in pairs(Utils:GetEnemyHeroesInRange(myHero.pos, R.range)) do
		local currentHero = Utils:GetEnemyHeroesInRange(myHero.pos, R.range)[_]

		if currentHero then
			if Utils:HasBuff(currentHero, "kaisapassivemarkerr") then
				Utils:CastSpell(HK_R, currentHero.pos:Extended(myHero.pos, 500), R.range)
			end
		end
	end
end

function Kaisa:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local defR = Utils:IsReady(_R) and Menu.Combo.rSettings.useR:Value() and Menu.Combo.rSettings.def.useRdefensive:Value() and 
		Utils:GetHpPercent(myHero) <= Menu.Combo.rSettings.def.useRdefensiveHp:Value() and 
			#Utils:GetEnemyHeroesInRange(myHero.pos, Menu.Combo.rSettings.def.useRdefensiveRange:Value()) > 0

	if not self.wTarget then return end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if defR then
		self:CastDefensiveR()
	end
end

function Kaisa:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)

	if not self.wTarget then return end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end
end

function Kaisa:OnDraw()

	local myHeroPos2D = myHero.pos:To2D()
	for _, v in pairs(Utils:GetEnemyHeroesInRange(myHero.pos, R.range)) do
		local currentHero = v
		if currentHero then
			if Utils:HasBuff(currentHero, "kaisapassivemarkerr") then
				local target2D = currentHero.pos:To2D()
				if Menu.Misc.drawLine:Value() then
					local width = Menu.Misc.lineWidth:Value()
					Draw.Line(myHeroPos2D.x, myHeroPos2D.y, target2D.x, target2D.y, width, Draw.Color(255, 155, 105, 240))
				end
			end
		end
	end
end

function Kaisa:GetWdmg(unit)

	local totalDmg = 0

	if unit then
		local wLvl = myHero:GetSpellData(_W).level
		if wLvl > 0 then

			local baseDmg = ({ 20, 45, 70, 95, 120  })[wLvl]
			local adDmg = myHero.totalDamage * 1.5
			local apDmg = myHero.ap * .45
			totalDmg = baseDmg + Utils:CalculateMagicalDamage(unit, baseDmg + adDmg + apDmg)
		end
	end
	return totalDmg
end

--[[                                                                       
                                                                        
        GGGGGGGGGGGGG                  lllllll   iiii                   
     GGG::::::::::::G                  l:::::l  i::::i                  
   GG:::::::::::::::G                  l:::::l   iiii                   
  G:::::GGGGGGGG::::G                  l:::::l                          
 G:::::G       GGGGGG  aaaaaaaaaaaaa    l::::l iiiiiii    ooooooooooo   
G:::::G                a::::::::::::a   l::::l i:::::i  oo:::::::::::oo 
G:::::G                aaaaaaaaa:::::a  l::::l  i::::i o:::::::::::::::o
G:::::G    GGGGGGGGGG           a::::a  l::::l  i::::i o:::::ooooo:::::o
G:::::G    G::::::::G    aaaaaaa:::::a  l::::l  i::::i o::::o     o::::o
G:::::G    GGGGG::::G  aa::::::::::::a  l::::l  i::::i o::::o     o::::o
G:::::G        G::::G a::::aaaa::::::a  l::::l  i::::i o::::o     o::::o
 G:::::G       G::::Ga::::a    a:::::a  l::::l  i::::i o::::o     o::::o
  G:::::GGGGGGGG::::Ga::::a    a:::::a l::::::li::::::io:::::ooooo:::::o
   GG:::::::::::::::Ga:::::aaaa::::::a l::::::li::::::io:::::::::::::::o
     GGG::::::GGG:::G a::::::::::aa:::al::::::li::::::i oo:::::::::::oo 
        GGGGGG   GGGG  aaaaaaaaaa  aaaalllllllliiiiiiii   ooooooooooo   
                                                                      
--]]

class "Galio"

function Galio:__init()

	if charName ~= "Galio" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Galio:LoadSpells()

	Q = { range = 825,	width = myHero:GetSpellData(_Q).width,	speed = myHero:GetSpellData(_Q).speed,	delay = 0.25 }
	W = { range = 350 }
	E = { range = 650, width = 160,		speed = 1400,	delay = 0.45,	collision = Collision:SetSpell(650, 1400, 0.45, 160, true)}
	R = { range = 4000 }
end

function Galio:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Misc ---
		Menu.Misc:MenuElement({id = "drawLine", name = "Draw line to low Allies", value = true })
		Menu.Misc:MenuElement({id = "drawHP", name = "HP % to draw line", value = 50, min = 1, max = 100, step = 1 })
end

function Galio:OnTick()

	if #Utils:GetEnemyHeroesInRange(myHero.pos, 250) > 0 then
		if Utils:HasBuff(myHero, "galiow") then
			Orb:DisableMovement(true)
		else
			Orb:DisableMovement(false)
		end
	else
		Orb:DisableMovement(false)
	end

	local rRvl = myHero:GetSpellData(_R).level
	R.range = ({ 4000, 4750, 5500 })[rRvl]

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Galio:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if Utils:GetDistance(myHero.pos, nPred) < Q.range then
					if nPred:To2D().onScreen then
						Utils:CastSpell(HK_Q, nPred, Q.range)
					end
				end
			end				
		else
			if t and aimPos then
				if Utils:GetDistance(myHero.pos, aimPos) < Q.range then
					if aimPos:To2D().onScreen then
						Utils:CastSpell(HK_Q, aimPos, Q.range)
					end
				end
			end
			if hitChance and aimPos2 and hitChance >= minAccuracy then
				if Utils:GetDistance(myHero.pos, aimPos2) < Q.range then
					if aimPos2:To2D().onScreen then
						Utils:CastSpell(HK_Q, aimPos2, Q.range)
					end
				end
			end
		end
	end
end

function Galio:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then 
		local pred = Utils:GetPred(unit, _huge, 0.4)
		if pred then
			if pred:To2D().onScreen then
				if Utils:GetDistance(pred, myHero.pos) < W.range then
					Control.KeyDown(HK_W)
				else
					if Utils:GetDistance(pred, myHero.pos) > W.range or Utils:GetDistance(unit.pos, myHero.pos) > W.range then
						Control.KeyUp(HK_W)
					end
				end
			end
		end
	else
		if Control.IsKeyDown(HK_W) then
			Control.KeyUp(HK_W)
		end
	end
end

function Galio:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, E.speed, E.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if Utils:GetDistance(nPred, myHero.pos) < E.range and nPred:To2D().onScreen then
					Utils:CastSpell(HK_E, myHero.pos:Extended(nPred, E.range), E.range)
				end
			end		
		else
			if t and aimPos then
				if Utils:GetDistance(aimPos, myHero.pos) < E.range and aimPos:To2D().onScreen then
					Utils:CastSpell(HK_E, myHero.pos:Extended(aimPos, E.range), E.range)
				end
			end
			if hitChance and aimPos2 and hitChance >= minAccuracy then 
				if Utils:GetDistance(aimPos2, myHero.pos) < E.range and aimPos2:To2D().onScreen then
					Utils:CastSpell(HK_E, myHero.pos:Extended(aimPos2, E.range), E.range)
				end
			end
		end
	end
end

function Galio:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Galio:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Galio:OnDraw()

	local myHeroPos2D = myHero.pos:To2D()
	if Utils:IsReady(_R) then
		if #Utils:GetAllyHeroesInRange(myHero.pos, R.range) > 0 then
			for k, v in pairs(Utils:GetAllyHeroesInRange(myHero.pos, R.range)) do
				local currentHero = Utils:GetAllyHeroesInRange(myHero.pos, R.range)[k]
				if currentHero then
					if currentHero ~= myHero then
						if Utils:GetHpPercent(currentHero) < Menu.Misc.drawHP:Value() then
							local ally2D = currentHero.pos:To2D()
							if Menu.Misc.drawLine:Value() then
								Draw.Line(myHeroPos2D.x, myHeroPos2D.y, ally2D.x, ally2D.y, 1, Draw.Color(255, 255, 0, 0))
							end
						end
					end
				end
			end
		end
	end
end

--[[                                                                                                     
                                                                                                        
TTTTTTTTTTTTTTTTTTTTTTT                                                                                 
T:::::::::::::::::::::T                                                                                 
T:::::::::::::::::::::T                                                                                 
T:::::TT:::::::TT:::::T                                                                                 
TTTTTT  T:::::T  TTTTTT    eeeeeeeeeeee        eeeeeeeeeeee       mmmmmmm    mmmmmmm      ooooooooooo   
        T:::::T          ee::::::::::::ee    ee::::::::::::ee   mm:::::::m  m:::::::mm  oo:::::::::::oo 
        T:::::T         e::::::eeeee:::::ee e::::::eeeee:::::eem::::::::::mm::::::::::mo:::::::::::::::o
        T:::::T        e::::::e     e:::::ee::::::e     e:::::em::::::::::::::::::::::mo:::::ooooo:::::o
        T:::::T        e:::::::eeeee::::::ee:::::::eeeee::::::em:::::mmm::::::mmm:::::mo::::o     o::::o
        T:::::T        e:::::::::::::::::e e:::::::::::::::::e m::::m   m::::m   m::::mo::::o     o::::o
        T:::::T        e::::::eeeeeeeeeee  e::::::eeeeeeeeeee  m::::m   m::::m   m::::mo::::o     o::::o
        T:::::T        e:::::::e           e:::::::e           m::::m   m::::m   m::::mo::::o     o::::o
      TT:::::::TT      e::::::::e          e::::::::e          m::::m   m::::m   m::::mo:::::ooooo:::::o
      T:::::::::T       e::::::::eeeeeeee   e::::::::eeeeeeee  m::::m   m::::m   m::::mo:::::::::::::::o
      T:::::::::T        ee:::::::::::::e    ee:::::::::::::e  m::::m   m::::m   m::::m oo:::::::::::oo 
      TTTTTTTTTTT          eeeeeeeeeeeeee      eeeeeeeeeeeeee  mmmmmm   mmmmmm   mmmmmm   ooooooooooo   
                                                                                                                                                                                                              
--]]

class "Teemo"

function Teemo:__init()

	if charName ~= "Teemo" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
end

function Teemo:LoadSpells()

	Q = { range = 680,	delay = 0.25 }
	W = { range = 500 }
	E = { }
	R = { }
end

function Teemo:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
end

function Teemo:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Teemo:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Utils:CastSpell(HK_Q, unit.pos, Q.range)
		end
	end
end

function Teemo:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		Control.CastSpell(HK_W)
	end
end

function Teemo:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end
end

function Teemo:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end
end

--[[
                                                                                                                                                 
                                                                                                                                                 
TTTTTTTTTTTTTTTTTTTTTTT                      iiii                            tttt                                                                
T:::::::::::::::::::::T                     i::::i                        ttt:::t                                                                
T:::::::::::::::::::::T                      iiii                         t:::::t                                                                
T:::::TT:::::::TT:::::T                                                   t:::::t                                                                
TTTTTT  T:::::T  TTTTTTrrrrr   rrrrrrrrr   iiiiiii     ssssssssss   ttttttt:::::ttttttt      aaaaaaaaaaaaa   nnnn  nnnnnnnn      aaaaaaaaaaaaa   
        T:::::T        r::::rrr:::::::::r  i:::::i   ss::::::::::s  t:::::::::::::::::t      a::::::::::::a  n:::nn::::::::nn    a::::::::::::a  
        T:::::T        r:::::::::::::::::r  i::::i ss:::::::::::::s t:::::::::::::::::t      aaaaaaaaa:::::a n::::::::::::::nn   aaaaaaaaa:::::a 
        T:::::T        rr::::::rrrrr::::::r i::::i s::::::ssss:::::stttttt:::::::tttttt               a::::a nn:::::::::::::::n           a::::a 
        T:::::T         r:::::r     r:::::r i::::i  s:::::s  ssssss       t:::::t              aaaaaaa:::::a   n:::::nnnn:::::n    aaaaaaa:::::a 
        T:::::T         r:::::r     rrrrrrr i::::i    s::::::s            t:::::t            aa::::::::::::a   n::::n    n::::n  aa::::::::::::a 
        T:::::T         r:::::r             i::::i       s::::::s         t:::::t           a::::aaaa::::::a   n::::n    n::::n a::::aaaa::::::a 
        T:::::T         r:::::r             i::::i ssssss   s:::::s       t:::::t    tttttta::::a    a:::::a   n::::n    n::::na::::a    a:::::a 
      TT:::::::TT       r:::::r            i::::::is:::::ssss::::::s      t::::::tttt:::::ta::::a    a:::::a   n::::n    n::::na::::a    a:::::a 
      T:::::::::T       r:::::r            i::::::is::::::::::::::s       tt::::::::::::::ta:::::aaaa::::::a   n::::n    n::::na:::::aaaa::::::a 
      T:::::::::T       r:::::r            i::::::i s:::::::::::ss          tt:::::::::::tt a::::::::::aa:::a  n::::n    n::::n a::::::::::aa:::a
      TTTTTTTTTTT       rrrrrrr            iiiiiiii  sssssssssss              ttttttttttt    aaaaaaaaaa  aaaa  nnnnnn    nnnnnn  aaaaaaaaaa  aaaa
                                                                                                                                             
--]]

class "Tristana"

function Tristana:__init()

	if charName ~= "Tristana" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
end

function Tristana:LoadSpells()

	Q = { range = 525 }
	W = { range = 900 }
	E = { range = 525 }
	R = { range = 525, knockbackRange = 600 }
end

function Tristana:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R finisher", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Misc --
		Menu.Misc:MenuElement({id = "drawInsec", name = "Draw flashy Insec lines", value = true})
end

function Tristana:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)
	
	self:UpdateRange()

	if not isEvading then
		if Orb.combo then
			self:ForceTarget()
			self:Combo()
			self:ReleaseTarget()
		elseif Orb.harass then
			self:Harass()
		elseif Orb.laneClear or Orb.jungleClear then
			self:ForceMinion()
			self:ReleaseMinions()
		end
	end
end

function Tristana:ForceTarget()

	for _,v in pairs(Utils:GetEnemyHeroesInRange(myHero.pos, E.range)) do
		local enemy = v
		if self:HasEbuff(v) then
			if _G.SDK then
				_G.SDK.Orbwalker.ForceTarget = v
			elseif _G.EOWLoaded then
				EOW:ForceTarget(v)
			elseif _G.GOS then
				GOS:ForceTarget(v)
			end
		end
	end
end

function Tristana:ForceMinion()

	for _,v in pairs(Utils:GetEnemyMinionsInRange(myHero.pos, E.range)) do
		local enemy = v
		if self:HasEbuff(v) then
			if _G.SDK then
				_G.SDK.Orbwalker.ForceTarget = v
			elseif _G.EOWLoaded then
				EOW:ForceTarget(v)
			elseif _G.GOS then
				GOS:ForceTarget(v)
			end
		end
	end
end

function Tristana:UpdateRange()
		
	local level = myHero.levelData.lvl
	local rLevel = myHero:GetSpellData(_R).level
	local range = 525 + (8 * (level - 1))
	local knockback = ({ 600,  800, 1000 })[rLevel]

	if E and E.range then
		E.range = range
	end
	if Q and Q.range then
		Q.range = range
	end
	if R and R.range and rLevel > 0 then
		R.range = range
	end
	R.knockbackRange = knockback
end

function Tristana:ReleaseTarget()

	if #Utils:GetEnemyHeroesInRange(myHero.pos, E.range) == 0 then
		if _G.SDK then
			_G.SDK.Orbwalker.ForceTarget = nil
		elseif _G.EOWLoaded then
			EOW:ForceTarget(nil)
		elseif _G.GOS then
			GOS:ForceTarget(nil)
		end
	end
end

function Tristana:ReleaseMinions()

	if #Utils:GetEnemyMinionsInRange(myHero.pos, E.range) == 0 then
		if _G.SDK then
			_G.SDK.Orbwalker.ForceTarget = nil
		elseif _G.EOWLoaded then
			EOW:ForceTarget(nil)
		elseif _G.GOS then
			GOS:ForceTarget(nil)
		end
	end
end

function Tristana:HasEbuff(unit)

	for i = 1, unit.buffCount do
		local buff = unit:GetBuff(i)
		if buff and buff.name == "tristanaechargesound" then
			return true
		end
	end
	return false
end

function Tristana:GetEstacks(unit)

	local stacks = 0
	if self:HasEbuff(unit) then
		for i = 1, unit.buffCount do
			local buff = unit:GetBuff(i)
			if buff and buff.count > 0 and buff.name:lower() == "tristanaecharge" then
				stacks = buff.count
			end
		end
	end
	return stacks
end

function Tristana:GetEdmg(unit, count)

	local total = 0
	local eLvl = myHero:GetSpellData(_E).level
	if eLvl > 0 then
		local raw = ({ 60, 70, 80, 90, 100 })[eLvl]
		local m = ({ .5, .6, .7, .8, .9 })[eLvl]
		local bonusDmg = m * myHero.bonusDamage
		total = raw + bonusDmg
		total = total + self:GetStackDmg(unit) * count
	end
	return Utils:CalculatePhysicalDamage(unit, total)
end

function Tristana:GetStackDmg(unit)

	local total = 0
	local eLvl = myHero:GetSpellData(_E).level
	if eLvl > 0 then
		local raw = ({ 18, 21, 24, 27, 30 })[eLvl]
		local m = ({ .15, .18, .21, .24, .27 })[eLvl]
		local bonusDmg = m * myHero.bonusDamage
		total = raw + bonusDmg 
	end
	return total
end

function Tristana:GetRdmg(unit)

	local total = 0
	local rLvl = myHero:GetSpellData(_R).level
	if rLvl > 0 then
		local raw = ({ 300, 400, 500 })[rLvl]
		local ap = myHero.ap

		total = raw + ap
	end
	return Utils:CalculateMagicalDamage(unit, total)
end

function Tristana:GetERdmg(unit, count)
	return self:GetEdmg(unit, count + 1) + self:GetRdmg(unit)
end

function Tristana:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Control.CastSpell(HK_Q)
		end
	end
end

function Tristana:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Control.CastSpell(HK_E, unit.pos)
		end
	end
end

function Tristana:CastRkill(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) then
		if (self:GetERdmg(unit, self:GetEstacks(unit)) > unit.health) or 
			self:GetRdmg(unit) > unit.health then
			Control.CastSpell(HK_R, unit)
		end
	end
end

function Tristana:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if self.rTarget then
		if rReady then
			self:CastRkill(self.rTarget)
		end
	end
end

function Tristana:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

--[[
                                                                                                                                   
          JJJJJJJJJJJ  iiii                                        
          J:::::::::J i::::i                                       
          J:::::::::J  iiii                                        
          JJ:::::::JJ                                              
            J:::::J  iiiiiii nnnn  nnnnnnnn    xxxxxxx      xxxxxxx
            J:::::J  i:::::i n:::nn::::::::nn   x:::::x    x:::::x 
            J:::::J   i::::i n::::::::::::::nn   x:::::x  x:::::x  
            J:::::j   i::::i nn:::::::::::::::n   x:::::xx:::::x   
            J:::::J   i::::i   n:::::nnnn:::::n    x::::::::::x    
JJJJJJJ     J:::::J   i::::i   n::::n    n::::n     x::::::::x     
J:::::J     J:::::J   i::::i   n::::n    n::::n     x::::::::x     
J::::::J   J::::::J   i::::i   n::::n    n::::n    x::::::::::x    
J:::::::JJJ:::::::J  i::::::i  n::::n    n::::n   x:::::xx:::::x   
 JJ:::::::::::::JJ   i::::::i  n::::n    n::::n  x:::::x  x:::::x  
   JJ:::::::::JJ     i::::::i  n::::n    n::::n x:::::x    x:::::x 
     JJJJJJJJJ       iiiiiiii  nnnnnn    nnnnnnxxxxxxx      xxxxxxx

--]]

class "Jinx"

function Jinx:__init()

	if charName ~= "Jinx" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Jinx:LoadSpells()

	attackRange = 525
	Q = { range = 600 }
	W = { range = 1450, speed = myHero:GetSpellData(_W).speed, width = 75,	delay = 0.5, 
		collision = Collision:SetSpell(1450, myHero:GetSpellData(_W).speed, 0.6, myHero:GetSpellData(_W).width, true) }
	E = { range = 900, speed = myHero:GetSpellData(_E).speed, width = 250,	delay = 0.5 }
	R = { range = 25000, speed = 1700, width = 140,	delay = 0.6 }
end

function Jinx:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useQtoggle", name = "Passive Logic Toggle", key = string.byte("T"), toggle = true })
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})
		Menu.Combo:MenuElement({id = "useKs", name = "KS everything!", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Jinx:OnTick()

	self.isQ1 = myHero:GetSpellData(_Q).toggleState == 1
	self:UpdateRange()
	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Jinx:UpdateRange()

	local qLvl = myHero:GetSpellData(_Q).level
	local range = ({800, 825, 850, 875, 900})[qLvl] -- -200 bear with me

	if qLvl > 0 then
		Q.range = range
	end
end

function Jinx:GetQstacks()

	for i = 1, myHero.buffCount do
		local buff = myHero:GetBuff(i)
		if buff then 
			if buff.count > 0 and buff.name:lower() == "jinxqramp" then 
				return buff.count
			end
		end
	end
end

function Jinx:CastQcombo(unit)

	local qStacks = self:GetQstacks() or 0
	if myHero.attackData.state == STATE_WINDUP then return end

	if self.isQ1 then
		if Utils:IsValid(unit, myHero.pos, Q.range) then
			if (qStacks == 3 and Menu.Combo.useQtoggle:Value()) or Utils:GetDistance(myHero.pos, unit.pos) > attackRange then
				Control.CastSpell(HK_Q)
			end
		end
	else
		if Utils:IsValid(unit, myHero.pos, attackRange) then
			if (qStacks < 3 or not Menu.Combo.useQtoggle:Value()) and myHero.totalDamage < unit.health then
				Control.CastSpell(HK_Q)
			end
		end
	end
end

function Jinx:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		if Utils:GetDistance(myHero.pos, unit.pos) < Q.range then return end
		if myHero.attackData.state ~= STATE_WINDUP then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, W.speed, W.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					local col, units = W.collision:__GetCollision(myHero.pos, nPred, 5)
					if not col then
						if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < W.range then
							Utils:CastSpell(HK_W, nPred, W.range)
						end					
					end
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					local col, units = W.collision:__GetCollision(myHero.pos, aimPos, 5)
					if not col then
						if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < W.range then
							Utils:CastSpell(HK_W, aimPos, W.range)
						end
					end
				end
				if hitChance and aimPos2 then
					if hitChance >= minAccuracy or (hitChance >= 1 and unit.health < self:GetWdmg(unit) and Menu.Combo.useKs:Value()) then
						local col, units = W.collision:__GetCollision(myHero.pos, aimPos2, 5)
						if not col then
							if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < W.range then
								Utils:CastSpell(HK_W, aimPos2, W.range)
							end
						end
					end
				end
			end
		end
	end
end

function Jinx:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, E.speed, E.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
						Utils:CastSpell(HK_E, nPred, E.range)
					end					
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
						Utils:CastSpell(HK_E, aimPos, E.range)
					end
				end
			end
		end
	end
end

function Jinx:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range * 0.5) then
		if self:GetRdmg(unit) > unit.health and Utils:GetDistance(unit.pos, myHero.pos) > Q.range then
			if myHero.attackData.state ~= STATE_WINDUP then
				local t, aimPos = HPred:GetReliableTarget(myHero.pos, R.range, R.delay, R.speed, R.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
				local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, R.range, R.delay, R.speed, R.width, false, unit.charName)
				local useNpred = Menu.AiO.Pred.pred:Value() == 1
				local nPred = Utils:GetPred(unit, R.speed, R.delay)
				local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
				if useNpred then
					if nPred then
						if Utils:GetDistance(nPred, myHero.pos) < R.range then
							if nPred:To2D().onScreen then
								Utils:CastSpell(HK_R, nPred, R.range)
							else
								Utils:CastSpellMM(HK_R, nPred, R.range)
							end
						end			
					end
				else
					if t and aimPos and t.networkID == unit.networkID then
						if Utils:GetDistance(aimPos, myHero.pos) < R.range then
							if aimPos:To2D().onScreen then
								Utils:CastSpell(HK_R, aimPos, R.range)
							else
								Utils:CastSpellMM(HK_R, aimPos, R.range)
							end
						end
					end
					if hitChance and aimPos2 then
						if hitChance >= minAccuracy or (hitChance >= 2 and unit.health < self:GetRdmg(unit) and Menu.Combo.useKs:Value()) then
							if Utils:GetDistance(aimPos2, myHero.pos) < R.range then
								if aimPos2:To2D().onScreen then
									Utils:CastSpell(HK_R, aimPos2, R.range)
								else
									Utils:CastSpellMM(HK_R, aimPos2, R.range)
								end
							end
						end
					end
				end
			end
		end
	end
end

function Jinx:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, Q.range) == 0 and not self.isQ1 then
			Control.CastSpell(HK_Q)
		end
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Jinx:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if qReady then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, Q.range) == 0 and not self.isQ1 then
			Control.CastSpell(HK_Q)
		end
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Jinx:GetWdmg(unit)
	
	local totalDmg = 0

	if unit then
		local wLvl = myHero:GetSpellData(_W).level
		if wLvl > 0 then
			local baseDmg = ({10, 60, 110, 160, 210})[wLvl]
			local adMod = myHero.totalDamage * 1.4

			totalDamage = baseDmg + adMod
		end
	end

	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Jinx:GetRdmg(unit)

	local totalDmg = 0

	if unit then

		local rLvl = myHero:GetSpellData(_R).level
		local baseDmg = ({250, 350, 450})[rLvl]
		local bonusAd = myHero.bonusDamage * 1.5
		if rLvl > 0 then
			local missingHp = ({0.25, 0.3, 0.35})[rLvl] * (unit.maxHealth - unit.health)
			local distance = Utils:GetDistance(myHero.pos, unit.pos)
			local maxDmg = baseDmg + bonusAd + missingHp
			local minDmg = maxDmg * 0.1

			if distance < 1500 then
				distance = distance
			else
				distance = 1500
			end

			totalDmg = minDmg + ((0.06 * minDmg) * (distance * 0.1))
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Jinx:OnDraw()

	local myHeroPos2D = myHero.pos:To2D()
	if not Menu.Combo.useQtoggle:Value() then
		Draw.Text("Q logic OFF", 12, myHeroPos2D.x - 20, myHeroPos2D.y + 50, Draw.Color(255,255,0,0))
	end
end


--[[                                                                                                                         
                                                                                                                         
        CCCCCCCCCCCCC                    iiii           tttt          lllllll                                            
     CCC::::::::::::C                   i::::i       ttt:::t          l:::::l                                            
   CC:::::::::::::::C                    iiii        t:::::t          l:::::l                                            
  C:::::CCCCCCCC::::C                                t:::::t          l:::::l                                            
 C:::::C       CCCCCC  aaaaaaaaaaaaa   iiiiiii ttttttt:::::ttttttt     l::::l yyyyyyy           yyyyyyynnnn  nnnnnnnn    
C:::::C                a::::::::::::a  i:::::i t:::::::::::::::::t     l::::l  y:::::y         y:::::y n:::nn::::::::nn  
C:::::C                aaaaaaaaa:::::a  i::::i t:::::::::::::::::t     l::::l   y:::::y       y:::::y  n::::::::::::::nn 
C:::::C                         a::::a  i::::i tttttt:::::::tttttt     l::::l    y:::::y     y:::::y   nn:::::::::::::::n
C:::::C                  aaaaaaa:::::a  i::::i       t:::::t           l::::l     y:::::y   y:::::y      n:::::nnnn:::::n
C:::::C                aa::::::::::::a  i::::i       t:::::t           l::::l      y:::::y y:::::y       n::::n    n::::n
C:::::C               a::::aaaa::::::a  i::::i       t:::::t           l::::l       y:::::y:::::y        n::::n    n::::n
 C:::::C       CCCCCCa::::a    a:::::a  i::::i       t:::::t    tttttt l::::l        y:::::::::y         n::::n    n::::n
  C:::::CCCCCCCC::::Ca::::a    a:::::a i::::::i      t::::::tttt:::::tl::::::l        y:::::::y          n::::n    n::::n
   CC:::::::::::::::Ca:::::aaaa::::::a i::::::i      tt::::::::::::::tl::::::l         y:::::y           n::::n    n::::n
     CCC::::::::::::C a::::::::::aa:::ai::::::i        tt:::::::::::ttl::::::l        y:::::y            n::::n    n::::n
        CCCCCCCCCCCCC  aaaaaaaaaa  aaaaiiiiiiii          ttttttttttt  llllllll       y:::::y             nnnnnn    nnnnnn
                                                                                    y:::::y                              
                                                                                   y:::::y                               
                                                                                  y:::::y                                
                                                                                 y:::::y                                 
                                                                                yyyyyyy                                  
                                                                                                                                                                                                                                                  
--]]

class "Caitlyn"

function Caitlyn:__init()

	if charName ~= "Caitlyn" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	lastTrap = GetTickCount()
end

function Caitlyn:LoadSpells()

	attackRange = 650
	Q = { range = 1250, 	speed = 2200, 	width = 90, 	delay = 0.625 	}
	W = { range = 800, 		speed = _huge, 	width = 75,		delay = 0.25 	}
	E = { range = 750, 		speed = 1500, 	width = 60,		delay = 0.25, 
		collision = Collision:SetSpell(750, 1500, 0.25, 60, true) }
	R = { range = 2000, 	speed = 3200,	collision = Collision:SetSpell(2000, 3200, 0.25, 75, false)}
end

function Caitlyn:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Caitlyn:OnTick()

	self:UpdateRange()
	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Caitlyn:UpdateRange()

	local rLvl = myHero:GetSpellData(_R).level
	local range = 2000
	if rLvl > 0 then
		range = ({2000, 2500, 3000})[rLvl]
		R.range = range
		R.collision = Collision:SetSpell(range, 3200, 0.25, 75, false)
	end
end

function Caitlyn:CastQcombo(unit)

	local rLvl = myHero:GetSpellData(_R).level
	local rMana = 0

	if not unit then return end

	if rLvl > 0 then
		rMana = myHero:GetSpellData(_R).mana
	end

	if myHero.attackData.state == STATE_WINDUP or (myHero.mana <= rMana and Utils:IsReady(_R)) then return end

	if Utils:IsValid(unit, myHero.pos, Q.range) and Utils:GetDistance(unit.pos, myHero.pos) > attackRange and not 
		Utils:HasBuff(unit, "caitlynyordletrapinternal") then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.range then
					Utils:CastSpell(HK_Q, nPred, Q.range)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < Q.range then
					Utils:CastSpell(HK_Q, aimPos, Q.range)
				end
			end
			if hitChance and aimPos2 then
				if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < E.range then
					if hitChance >= minAccuracy or (hitChance >= 1 and self:GetComboDmg(unit) > unit.health) then
						Utils:CastSpell(HK_Q, aimPos2, Q.range)
					end
				end
			end
		end
	end
end

function Caitlyn:CastWcombo(unit)

	if not unit then return end

	if myHero.attackData.state == STATE_WINDUP or lastTrap + 2500 > GetTickCount() then return end

	if Utils:IsValid(unit, myHero.pos, W.range) then
		if Utils:HasBuff(unit, "caitlynyordletrapinternal") then return end
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, .1, false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, nPred, W.range)
					lastTrap = GetTickCount()
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, aimPos, W.range)
					lastTrap = GetTickCount()
				end
			end
			if hitChance and aimPos2 and hitChance > 3 then
				if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, aimPos2, W.range)
					lastTrap = GetTickCount()
				end
			end
		end
	end
end

function Caitlyn:CastEcombo(unit)

	if not unit then return end

	if Utils:IsValid(unit, myHero.pos, E.range) then
		if myHero.attackData.state ~= STATE_WINDUP and not Utils:HasBuff(unit, "caitlynyordletrapinternal") then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, E.speed, E.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					local col, units = E.collision:__GetCollision(myHero.pos, nPred, 5)
					if not col then
						if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
							Utils:CastSpell(HK_E, nPred, E.range)
						end					
					end
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					local col, units = E.collision:__GetCollision(myHero.pos, aimPos, 5)
					if not col then
						if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
							Utils:CastSpell(HK_E, aimPos, E.range)
						end
					end
				end
				if hitChance and aimPos2 then
					local col, units = E.collision:__GetCollision(myHero.pos, aimPos2, 5)
					if aimPos2:To2D().onScreen and not col and Utils:GetDistance(aimPos2, myHero.pos) < E.range then
						if hitChance >= minAccuracy or (hitChance >= 1 and self:GetComboDmg(unit) > unit.health) then
							Utils:CastSpell(HK_E, aimPos2, E.range)
						end
					end
				end
			end
		end
	end
end

function Caitlyn:CastRcombo(unit)

	if not unit then return end

	if Utils:IsValid(unit, myHero.pos, R.range) then
		if myHero.attackData.state ~= STATE_WINDUP and Utils:GetDistance(unit.pos, myHero.pos) > Q.range and self:GetRdmg(unit) > unit.health then
			local col, units = R.collision:__GetHeroCollision(myHero.pos, unit.pos, 3, unit)
			if not col then
				if Utils:GetDistance(unit.pos, myHero.pos) < R.range then
					if unit.pos:To2D().onScreen then
						Control.CastSpell(HK_R, unit)
					else
						Utils:CastSpellMM(HK_R, unit.pos, R.range)
					end
				end
			end
		end
	end
end

function Caitlyn:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if myHero.attackData.state ~= STATE_WINDUP and Orb.canAttack then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2) > 0 then
			for i=1, #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2) do
				local enemy = Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2)[i]

				if enemy then
					if Utils:IsValid(enemy, myHero.pos, attackRange * 2) then
						if Utils:HasBuff(enemy, "caitlynyordletrapinternal") then
							Control.Attack(enemy)
							break
						end
					end
				end
			end
		end
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Caitlyn:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if myHero.attackData.state ~= STATE_WINDUP then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2) > 0 then
			for i=1, #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2) do
				local enemy = Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2)[i]

				if enemy then
					if Utils:IsValid(enemy, myHero.pos, attackRange * 2) then
						if Utils:HasBuff(enemy, "caitlynyordletrapinternal") then
							Control.Attack(enemy)
						end
					end
				end
			end
		end
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Caitlyn:GetRdmg(unit)

	local totalDmg = 0

	if unit then
		local rLvl = myHero:GetSpellData(_R).level
		if rLvl > 0 then
			local baseDmg = ({ 250, 475, 700 })[rLvl]
			local bonusAd = myHero.bonusDamage * 2
			totalDmg = baseDmg + bonusAd
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Caitlyn:GetQdmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 30, 70, 110, 150, 190 })[qLvl]
			local bonusAd = ({ 1.3, 1.4, 1.5, 1.6, 1.7 })[qLvl]
			totalDmg = baseDmg + (bonusAd * myHero.totalDamage)
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Caitlyn:GetComboDmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		local rLvl = myHero:GetSpellData(_R).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		if rLvl > 0 then
			totalDmg = totalDmg + self:GetRdmg(unit)
		end
			totalDmg = totalDmg + myHero.totalDamage * 1.5
	end
	return totalDmg
end

--[[                                                                                 
                                                                                 
KKKKKKKKK    KKKKKKK                                                             
K:::::::K    K:::::K                                                             
K:::::::K    K:::::K                                                             
K:::::::K   K::::::K                                                             
KK::::::K  K:::::KKK  aaaaaaaaaaaaa   yyyyyyy           yyyyyyynnnn  nnnnnnnn    
  K:::::K K:::::K     a::::::::::::a   y:::::y         y:::::y n:::nn::::::::nn  
  K::::::K:::::K      aaaaaaaaa:::::a   y:::::y       y:::::y  n::::::::::::::nn 
  K:::::::::::K                a::::a    y:::::y     y:::::y   nn:::::::::::::::n
  K:::::::::::K         aaaaaaa:::::a     y:::::y   y:::::y      n:::::nnnn:::::n
  K::::::K:::::K      aa::::::::::::a      y:::::y y:::::y       n::::n    n::::n
  K:::::K K:::::K    a::::aaaa::::::a       y:::::y:::::y        n::::n    n::::n
KK::::::K  K:::::KKKa::::a    a:::::a        y:::::::::y         n::::n    n::::n
K:::::::K   K::::::Ka::::a    a:::::a         y:::::::y          n::::n    n::::n
K:::::::K    K:::::Ka:::::aaaa::::::a          y:::::y           n::::n    n::::n
K:::::::K    K:::::K a::::::::::aa:::a        y:::::y            n::::n    n::::n
KKKKKKKKK    KKKKKKK  aaaaaaaaaa  aaaa       y:::::y             nnnnnn    nnnnnn
                                            y:::::y                              
                                           y:::::y                               
                                          y:::::y                                
                                         y:::::y                                 
                                        yyyyyyy                                  
                                                                                                                                                               
--]]

class "Kayn"

function Kayn:__init()

	if charName ~= "Kayn" then return end

	self.rangeUpdated = false
	self.lastParticleCheck = GetTickCount()

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Kayn:LoadSpells()

	Q = { range = 350,	width = 100, delay = 0.25, speed = 1000 }
	W = { range = myHero:GetSpellData(_W).range, width = 175, delay = 0.25, speed = 1750 }
	E = { range = 400 }
	R = { range = 550 }
end

function Kayn:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})
		Menu.Combo:MenuElement({id = "useRmyHp", name = "Use R if my HP < %", value = 50, min = 0, max = 100, step = 1})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
end

function Kayn:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.rTarget = Utils:GetTarget(R.range)

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		elseif Orb.laneClear or Orb.jungleClear then
			self:Clear()
		end
	end
end

function Kayn:CastQcombo(unit)

	if not unit then return end
	if myHero.attackData.state == STATE_WINDUP then return end

	local qWrange = Q.range + W.range
	local qRange = Q.range + Q.range

	if Utils:IsValid(unit, myHero.pos, qRange) or (Utils:IsValid(unit, myHero.pos, qWrange) and unit.health < self:GetComboDmg(unit)) then
		if myHero.attackData.state ~= STATE_WINDUP then
			local pred = Utils:GetPred(unit, Q.speed, Q.delay)
			if pred then
				Utils:CastSpell(HK_Q, pred, Q.range + W.range)
			end
		end
	end
end

function Kayn:CastWcombo(unit)

	if not unit then return end

	if myHero.attackData.state == STATE_WINDUP then return end

	if Utils:IsValid(unit, myHero.pos, W.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local hitChance2, aimPos3 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, nPred, W.range)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, aimPos, W.range)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy or (hitChance >= 1 and self:GetComboDmg(unit) >= unit.health) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < W.range then
						Utils:CastSpell(HK_W, aimPos2, W.range)	
					end
				end
			end
		end
	end
end

function Kayn:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) then
		if Utils:IsParticleInRange("kayn_base_primary_r_mark", unit.pos, unit.boundingRadius) then
			if self:GetComboDmg(unit) >= unit.health or Utils:GetHpPercent(myHero) < Menu.Combo.useRmyHp:Value() then
				Control.CastSpell(HK_R, unit)
			end
		end
	end
end

function Kayn:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end
	if #Utils:GetEnemyHeroesInRange(myHero.pos, R.range) > 0 then
		for i=1, #Utils:GetEnemyHeroesInRange(myHero.pos, R.range) do
			local enemy = Utils:GetEnemyHeroesInRange(myHero.pos, R.range)[i]

			if enemy and rReady  then
				self:CastRcombo(enemy)
			end
		end
	end
end

function Kayn:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end
end

function Kayn:Clear()
	
	local qReady = Menu.Clear.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Clear.useW:Value() and Utils:IsReady(_W)

	for i = 1, LocalGameMinionCount() do
		local m = LocalGameMinion(i)
		if m and Utils:IsValid(m, myHero.pos, W.range) then
			if wReady then
				Utils:CastSpell(HK_W, m.pos, W.range)
			end
			if qReady and Utils:GetDistance(m.pos, myHero.pos) <= Q.range + Q.width then
				Utils:CastSpell(HK_Q, m.pos, Q.range)
			end
		end
	end
end

function Kayn:GetQdmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 60, 80, 100, 120, 140 })[qLvl]
			local modifier = 0.65
			totalDmg = baseDmg + (modifier * myHero.bonusDamage)
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Kayn:GetWdmg(unit)

	local totalDmg = 0

	if unit then
		local wLvl = myHero:GetSpellData(_W).level
		if wLvl > 0 then
			local baseDmg = ({ 90, 135, 180, 225, 270 })[wLvl]
			local modifier = 1.3
			totalDmg = baseDmg + (modifier * myHero.bonusDamage)
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Kayn:GetRdmg(unit)

	local totalDmg = 0

	if unit then
		local rLvl = myHero:GetSpellData(_R).level
		if rLvl > 0 then
			local baseDmg = ({ 150, 250, 350 })[rLvl]
			local bonusAd = 1.75
			totalDmg = baseDmg + (bonusAd * myHero.bonusDamage)
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Kayn:GetComboDmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		local wLvl = myHero:GetSpellData(_W).level
		local rLvl = myHero:GetSpellData(_R).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		if wLvl > 0 then
			totalDmg = totalDmg + self:GetWdmg(unit)
		end
		if rLvl > 0 then
			totalDmg = totalDmg + self:GetRdmg(unit)
		end
		totalDmg = totalDmg + myHero.totalDamage
	end
	return totalDmg
end

function Kayn:ParticleHandler()

	for i = 1, LocalGameParticleCount() do
		local particle = LocalGameParticle(i);
		if particle and not particle.dead then
			local pos2D = particle.pos:To2D()
			local myHero2D = myHero.pos:To2D()
			local pos = particle.pos
			if Utils:GetDistance(pos, myHero.pos) <= 1000 then
				if pos2D.onScreen then
					if  _find(particle.name:lower(), "orb") and Utils:GetDistance(pos, myHero.pos) < 1000 then
						if _find(particle.name:lower(), "assassin") then
							Draw.Circle(pos, 40, 2, Draw.Color(150, 0, 100, 255))
						end
						if _find(particle.name:lower(), "slayer") then
							Draw.Circle(pos,40, 2, Draw.Color(150, 255, 100, 0))
						end
					else
						if self.rangeUpdated and _find(particle.name:lower(), "base_assassin") and Utils:GetDistance(pos, myHero.pos) < 50 then					
							self.rangeUpdated = true
							W.range = myHero:GetSpellData(_W).range
							W.delay = 0
						end
					end
				end
			end
		end
	end
end

function Kayn:OnDraw()

	if self.lastParticleCheck + 100 < GetTickCount() then
		self:ParticleHandler()
		self.lastParticleCheck = GetTickCount()
	end
end

--[[                                                                                                                                                             
                                                                                                                                                              
MMMMMMMM               MMMMMMMM                                            tttt                                                  YYYYYYY       YYYYYYY  iiii  
M:::::::M             M:::::::M                                         ttt:::t                                                  Y:::::Y       Y:::::Y i::::i 
M::::::::M           M::::::::M                                         t:::::t                                                  Y:::::Y       Y:::::Y  iiii  
M:::::::::M         M:::::::::M                                         t:::::t                                                  Y::::::Y     Y::::::Y        
M::::::::::M       M::::::::::M  aaaaaaaaaaaaa       ssssssssss   ttttttt:::::ttttttt        eeeeeeeeeeee    rrrrr   rrrrrrrrr   YYY:::::Y   Y:::::YYYiiiiiii 
M:::::::::::M     M:::::::::::M  a::::::::::::a    ss::::::::::s  t:::::::::::::::::t      ee::::::::::::ee  r::::rrr:::::::::r     Y:::::Y Y:::::Y   i:::::i 
M:::::::M::::M   M::::M:::::::M  aaaaaaaaa:::::a ss:::::::::::::s t:::::::::::::::::t     e::::::eeeee:::::eer:::::::::::::::::r     Y:::::Y:::::Y     i::::i 
M::::::M M::::M M::::M M::::::M           a::::a s::::::ssss:::::stttttt:::::::tttttt    e::::::e     e:::::err::::::rrrrr::::::r     Y:::::::::Y      i::::i 
M::::::M  M::::M::::M  M::::::M    aaaaaaa:::::a  s:::::s  ssssss       t:::::t          e:::::::eeeee::::::e r:::::r     r:::::r      Y:::::::Y       i::::i 
M::::::M   M:::::::M   M::::::M  aa::::::::::::a    s::::::s            t:::::t          e:::::::::::::::::e  r:::::r     rrrrrrr       Y:::::Y        i::::i 
M::::::M    M:::::M    M::::::M a::::aaaa::::::a       s::::::s         t:::::t          e::::::eeeeeeeeeee   r:::::r                   Y:::::Y        i::::i 
M::::::M     MMMMM     M::::::Ma::::a    a:::::a ssssss   s:::::s       t:::::t    tttttte:::::::e            r:::::r                   Y:::::Y        i::::i 
M::::::M               M::::::Ma::::a    a:::::a s:::::ssss::::::s      t::::::tttt:::::te::::::::e           r:::::r                   Y:::::Y       i::::::i
M::::::M               M::::::Ma:::::aaaa::::::a s::::::::::::::s       tt::::::::::::::t e::::::::eeeeeeee   r:::::r                YYYY:::::YYYY    i::::::i
M::::::M               M::::::M a::::::::::aa:::a s:::::::::::ss          tt:::::::::::tt  ee:::::::::::::e   r:::::r                Y:::::::::::Y    i::::::i
MMMMMMMM               MMMMMMMM  aaaaaaaaaa  aaaa  sssssssssss              ttttttttttt      eeeeeeeeeeeeee   rrrrrrr                YYYYYYYYYYYYY    iiiiiiii
                                                                                                                                                                                                                                                                                                                          
--]]

class "MasterYi"

function MasterYi:__init()

	if charName ~= "MasterYi" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)

	if _G.SDK then
        _G.SDK.Orbwalker:OnPostAttack(function() self:OnPostAttack() end)
    elseif _G.EOW then
        EOW:AddCallback(EOW.AfterAttack, function() self:OnPostAttack() end)
    elseif _G.GOS then
        GOS:OnAttackComplete(function() self:OnPostAttack() end)
    end
end

function MasterYi:LoadSpells()

	attackRange = myHero.range + myHero.boundingRadius
	Q = { range = myHero:GetSpellData(_Q).range,	delay = 0.25 }
	W = { range = 500 }
	E = { }
	R = { }
end

function MasterYi:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "auto", name = "AutoWin", value = true, tooltip = "Kappa"})
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function MasterYi:OnTick()

	self.qTarget 	= Utils:GetTarget(Q.range)
	self.eTarget 	= Utils:GetTarget(Q.range)

	if #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange) > 0 or myHero.attackData.state == STATE_WINDUP then
		Orb:DisableMovement(true)
	else
		Orb:DisableMovement(false)
	end

	if not myHero.dead then
		if not isEvading then
			if Orb.combo then
				self:Combo()		
			end
		end
	end
end

function MasterYi:OnPostAttack()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	if (Orb.combo or Orb.harass) and qReady then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2) > 0 then
			for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2) do
				local current = Utils:GetEnemyHeroesInRange(myHero.pos, attackRange * 2)[i]
				if current then
					if Utils:IsValid(current, myHero.pos, Q.range) then
						Control.CastSpell(HK_Q, current)
						break
					end
				end
			end
		end
	end
end

function MasterYi:CastQcombo(unit)
	
	if unit and Utils:IsValid(unit, myHero.pos, Q.range) then
		if Utils:GetDistance(unit.pos, myHero.pos) > attackRange then
			Control.CastSpell(HK_Q, unit)
		end
	end
end

function MasterYi:CastQdodge()
	
	for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, 1000) do
		local current = Utils:GetEnemyHeroesInRange(myHero.pos, 1000)[i]
		if current then
			if current.activeSpell and current.activeSpell.valid and
				(current.activeSpell.target == myHero.handle or 
					Utils:GetDistance(current.activeSpell.placementPos, myHero.pos) <= myHero.boundingRadius * 2 + current.activeSpell.width) and not
				_find(current.activeSpell.name:lower(), "attack") then
				local startPos = current.activeSpell.startPos
				local placementPos = current.activeSpell.placementPos
				local width = 0
				if current.activeSpell.width > 0 then
					width = current.activeSpell.width
				else
					width = 100
				end
				local distance = Utils:GetDistance(myHero.pos, placementPos)											
				if current.activeSpell.target == myHero.handle then
					self:Qdodge()
					return
				else
					if distance <= width * 2 + myHero.boundingRadius then
						self:Qdodge()
						break						
					end
				end
			end
		end
	end
end

function MasterYi:Qdodge()
	
	for _, v in pairs(Utils:GetEnemyHeroesInRange(myHero.pos, Q.range)) do
		local hero = v
		if hero then
			if Utils:IsValid(hero, myHero.pos, Q.range) then
				Control.CastSpell(HK_Q, hero)
				return
			end
		end
	end

	for _, v in pairs(Utils:GetEnemyMinionsInRange(myHero.pos, Q.range)) do
		local minion = v
		if minion then
			if Utils:IsValid(minion, myHero.pos, Q.range) then
				Control.CastSpell(HK_Q, minion)
				break
			end
		end
	end
end

function MasterYi:CastEcombo(unit)

	if #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange) > 0 then
		for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, attackRange) do
			local current = Utils:GetEnemyHeroesInRange(myHero.pos, attackRange)[i]
			if current then
				if Utils:IsValid(current, myHero.pos, attackRange) then
					Control.CastSpell(HK_E)
					break
				end
			end
		end
	end
end

function MasterYi:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if qReady then
		self:CastQdodge()
		if self.qTarget then
			self:CastQcombo(self.qTarget)
		end
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function MasterYi:GetQdmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 25, 60, 95, 130, 165 })[qLvl]
			totalDmg = baseDmg + myHero.totalDamage
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function MasterYi:GetComboDmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		totalDmg = totalDmg + myHero.totalDamage
	end
	return totalDmg
end

--[[
                                                                                                                        
                                                                          dddddddd                                      
   SSSSSSSSSSSSSSS                                                        d::::::d                                      
 SS:::::::::::::::S                                                       d::::::d                                      
S:::::SSSSSS::::::S                                                       d::::::d                                      
S:::::S     SSSSSSS                                                       d:::::d                                       
S:::::S            yyyyyyy           yyyyyyynnnn  nnnnnnnn        ddddddddd:::::d rrrrr   rrrrrrrrr     aaaaaaaaaaaaa   
S:::::S             y:::::y         y:::::y n:::nn::::::::nn    dd::::::::::::::d r::::rrr:::::::::r    a::::::::::::a  
 S::::SSSS           y:::::y       y:::::y  n::::::::::::::nn  d::::::::::::::::d r:::::::::::::::::r   aaaaaaaaa:::::a 
  SS::::::SSSSS       y:::::y     y:::::y   nn:::::::::::::::nd:::::::ddddd:::::d rr::::::rrrrr::::::r           a::::a 
    SSS::::::::SS      y:::::y   y:::::y      n:::::nnnn:::::nd::::::d    d:::::d  r:::::r     r:::::r    aaaaaaa:::::a 
       SSSSSS::::S      y:::::y y:::::y       n::::n    n::::nd:::::d     d:::::d  r:::::r     rrrrrrr  aa::::::::::::a 
            S:::::S      y:::::y:::::y        n::::n    n::::nd:::::d     d:::::d  r:::::r             a::::aaaa::::::a 
            S:::::S       y:::::::::y         n::::n    n::::nd:::::d     d:::::d  r:::::r            a::::a    a:::::a 
SSSSSSS     S:::::S        y:::::::y          n::::n    n::::nd::::::ddddd::::::dd r:::::r            a::::a    a:::::a 
S::::::SSSSSS:::::S         y:::::y           n::::n    n::::n d:::::::::::::::::d r:::::r            a:::::aaaa::::::a 
S:::::::::::::::SS         y:::::y            n::::n    n::::n  d:::::::::ddd::::d r:::::r             a::::::::::aa:::a
 SSSSSSSSSSSSSSS          y:::::y             nnnnnn    nnnnnn   ddddddddd   ddddd rrrrrrr              aaaaaaaaaa  aaaa
                         y:::::y                                                                                        
                        y:::::y                                                                                         
                       y:::::y                                                                                          
                      y:::::y                                                                                           
                     yyyyyyy                                                                                            
                                                                                                                        
                                                                                                                        
--]]

class "Syndra"

function Syndra:__init()

	if charName ~= "Syndra" then return end

	self.lastParticleCheck = GetTickCount()
	self.lastCast = GetTickCount()
	self:LoadSpells()
	self:LoadMenu()

	_Balls = {}
	self.wState = 1
	self.rangeUpdated = false

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Syndra:LoadSpells()

	Q 	= 	{ range = 800,	delay = 0.25, 		width = 175,		speed = 1750 }
	W 	= 	{ range = 925, 	delay = 0.25, 		width = 225, 		speed = 1450 }
	QE 	= 	{ range = 1100, delay = 0.25, 		width = 150, 		speed = 2500 }
	E 	= 	{ range = 700, 	delay = 0.25,		width = 150,		speed = 2500 }
	R 	= 	{ range = 675, 	delay = 0.25, 		speed = _huge }
end

function Syndra:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Misc ---
		Menu.Misc:MenuElement({id = "drawings", name = "Fancy Drawings", value = true})
end

function Syndra:OnTick() 

	self.wState = myHero:GetSpellData(_W).toggleState 
	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)
	self.qEtarget = Utils:GetTarget(QE.range)
	
	if not self.rangeUpdated then
		if myHero.levelData.lvl > 6 then
			R.range = 750
			self.rangeUpdated = true
		end
	end

	if myHero.alive then
		if self.qTarget then
			if Utils:IsReady(_Q) then
				Orb:DisableAttacks(true)
			else
				Orb:DisableAttacks(false)
			end
		else
			Orb:DisableAttacks(false)
		end

		if self.lastParticleCheck + 500 < GetTickCount() then
			self:GetBalls()
		end

		if Orb.combo and not isEvading then
			self:Combo()
		elseif Orb.harass and not isEvading then
			self:Harass()
		end
	end
end

function Syndra:CastQcombo(unit)

	if unit then
		local qRange = Q.range + Q.width
		if Utils:IsValid(unit, myHero.pos, qRange) then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, qRange, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, qRange, Q.delay, Q.speed, Q.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) <= qRange then
						Utils:CastSpell(HK_Q, nPred, Q.range)
						self.lastCast = GetTickCount()
						return
					end					
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) <= qRange then
						Utils:CastSpell(HK_Q, aimPos, Q.range)
						self.lastCast = GetTickCount()
						return
					end
				end
				if hitChance and aimPos2 then
					if hitChance >= minAccuracy or (hitChance >= 1 and self:GetComboDmg(unit) >= unit.health) then
						if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < qRange then
							Utils:CastSpell(HK_Q, aimPos2, Q.range)
							self.lastCast = GetTickCount()
						end
					end
				end
			end
		end
	end
end

function Syndra:CastW1combo(unit)
		
	for _, ball in pairs(_Balls) do
		if ball then
			if ball.object.valid and ball.pos:To2D().onScreen then
				if Utils:GetDistance(ball.pos, myHero.pos) <= W.range then 
					Control.CastSpell(HK_W, ball.pos)
					self.lastCast = GetTickCount()
					return
				end
			end
		end
	end

	if #Utils:GetEnemyMinionsInRange(myHero.pos, W.range) > 0 then
		for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, W.range) do
			local minion = Utils:GetEnemyMinionsInRange(myHero.pos, W.range)[i]
			if minion then
				if Utils:IsValid(minion, myHero.pos, W.range) and minion.pos:To2D().onScreen then
					Control.CastSpell(HK_W, minion)
					self.lastCast = GetTickCount()
					break
				end
			end
		end
	end
end

function Syndra:CastW2combo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) <= W.range then
					Utils:CastSpell(HK_W, nPred, W.range)
					self.lastCast = GetTickCount()
					return
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) <= W.range then
					Utils:CastSpell(HK_W, aimPos, W.range)
					self.lastCast = GetTickCount()
					return
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy or (hitChance >= 1 and self:GetComboDmg(unit) >= unit.health) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) <= W.range then
						Utils:CastSpell(HK_W, aimPos2, W.range)
						self.lastCast = GetTickCount()	
					end
				end
			end
		end
	end
end

function Syndra:CastEcombo(unit)

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)

	if Utils:GetDistance(unit.pos, myHero.pos) > Q.range + Q.width and Utils:GetDistance(unit.pos, myHero.pos) < QE.range and qReady then
		local posExtended = myHero.pos:Extended(unit.pos, E.range)
		Utils:CastSpell(HK_Q, posExtended, E.range)
	end

	if Utils:IsValid(unit, myHero.pos, E.range * .75) then
		Utils:CastSpell(HK_E, unit.pos, E.range)
	end
end

function Syndra:CastQecombo(unit)

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)

	if Utils:IsValid(unit, myHero.pos, QE.range) then
		if Utils:GetDistance(unit.pos, myHero.pos) > Q.range + Q.width and Utils:GetDistance(unit.pos, myHero.pos) < QE.range and qReady then
			local posExtended = myHero.pos:Extended(unit.pos, E.range)
			Utils:CastSpell(HK_Q, posExtended, E.range)
		end

		for _, ball in pairs(_Balls) do
			if ball then
				if Utils:GetDistance(ball.pos, myHero.pos) <= E.range then
					local distance = QE.range
					local ballPosExtended = myHero.pos:Extended(ball.pos, distance)
					local radius = ball.object.boundingRadius
					for i = radius, distance, radius do
						local current = myHero.pos:Extended(ballPosExtended, i)
						if Menu.Misc.drawings:Value() then
							Draw.Circle(current, ball.object.boundingRadius, 2, Draw.Color(254, 138,43,226))
						end
						if Utils:GetDistance(unit.pos, current) <= radius + unit.boundingRadius then
							Utils:CastSpell(HK_E, ball.pos, E.range)
						end
					end
				end
			end
		end
	end
end

function Syndra:CastRcombo(unit)
	
	if Utils:IsValid(unit, myHero.pos, R.range + 200) then
		if unit.health <= self:GetRdmg(unit) and Utils:GetDistance(unit.pos, myHero.pos) <= R.range then
			Control.CastSpell(HK_R, unit)
		end
	end
end

function Syndra:Combo()

	local qReady 	= 	Menu.Combo.useQ:Value() and Utils:IsReady(_Q) and self.lastCast + 100 < GetTickCount()
	local wReady 	= 	Menu.Combo.useW:Value() and Utils:IsReady(_W) and self.lastCast + 100 < GetTickCount() and self.wState == 1
	local w2Ready 	= 	Menu.Combo.useW:Value() and Utils:IsReady(_W) and self.lastCast + 500 < GetTickCount() and self.wState == 2
	local eReady 	= 	Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady 	= 	Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end
	if not eReady then
		if wReady and self.wTarget then
			self:CastW1combo(self.wTarget)
		end
		if w2Ready and self.wTarget then
			self:CastW2combo(self.wTarget)
		end
	end
	if eReady then
		if self.qEtarget then
			self:CastQecombo(self.qEtarget)
			return
		else
			if self.eTarget then
				self:CastECombo(self.eTarget)
			end
		end
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Syndra:Harass()

	local qReady 	= 	Menu.Harass.useQ:Value() and Utils:IsReady(_Q) and self.lastCast + 100 < GetTickCount()
	local wReady 	= 	Menu.Harass.useW:Value() and Utils:IsReady(_W) and self.lastCast + 100 < GetTickCount() and self.wState == 1
	local w2Ready 	= 	Menu.Harass.useW:Value() and Utils:IsReady(_W) and self.lastCast + 100 < GetTickCount() and self.wState == 2
	local eReady 	= 	Menu.Harass.useE:Value() and Utils:IsReady(_E) and self.lastCast + 100 < GetTickCount()

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end
	if not eReady then
		if wReady and self.wTarget then
			self:CastW1combo(self.wTarget)
		end
		if w2Ready and self.wTarget then
			self:CastW2combo(self.wTarget)
		end
	end
	if eReady then
		if self.qEtarget then
			self:CastQecombo(self.qEtarget)
			return
		else
			if self.eTarget then
				self:CastECombo(self.eTarget)
			end
		end
	end
end

function Syndra:GetBalls()
	
	_balls = {}

	if self.lastParticleCheck + 50 > GetTickCount() then return end
	for i = 1, LocalGameObjectCount() do
		local object = LocalGameObject(i)
		if object and object.valid then
			local name = object.charName
			if name:lower() == "syndrasphere" then
				if not _balls[object.networkID] then
					local objectPos = object.pos
					_balls[object.networkID] = {}
					_balls[object.networkID]["object"] = object
					_balls[object.networkID]["pos"] = objectPos
				end
			end
		end
	end
	_Balls = _balls
	self.lastParticleCheck = GetTickCount()
end

function Syndra:GetComboDmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		local wLvl = myHero:GetSpellData(_W).level
		local eLvl = myHero:GetSpellData(_E).level
		local rLvl = myHero:GetSpellData(_R).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		if wLvl > 0 then
			totalDmg = totalDmg + self:GetWdmg(unit)
		end
		if eLvl > 0 then
			totalDmg = totalDmg + self:GetEdmg(unit)
		end
		if rLvl > 0 then
			totalDmg = totalDmg + self:GetRdmg(unit)
		end
	end
	return totalDmg
end

function Syndra:GetQdmg(unit)
	return 0
end

function Syndra:GetWdmg(unit)
	return 0
end

function Syndra:GetEdmg(unit)
	return 0
end


function Syndra:GetRdmg(unit)

	if not unit then return 0 end
	local totalDmg = 0
	local ballCount = 3
	if #_Balls > 0 then
		ballCount = ballCount + #_Balls
	end
	if ballCount > 7 then ballCount = 7 end
	if unit then
		local rLvl = myHero:GetSpellData(_R).level
		if rLvl > 0 then
			local baseDmg = ({ 90, 135, 180 })[rLvl]
			local ap = 0.2 * myHero.ap
			totalDmg = (baseDmg + ap) * ballCount
		end
	end
	return Utils:CalculateMagicalDamage(unit, totalDmg)
end


function Syndra:OnDraw()

	if Menu.Misc.drawings:Value() then
		for _, ball in pairs(_Balls) do
			if ball then
				Draw.Circle(ball.pos, ball.object.boundingRadius, 2, Draw.Color(255, 138,43,226))
				if Utils:IsReady(_E) and Utils:GetDistance(ball.pos, myHero.pos) <= E.range then
					local ballPosExtended = myHero.pos:Extended(ball.pos, QE.range)
					Draw.Line(ball.pos:To2D().x, ball.pos:To2D().y, ballPosExtended:To2D().x, ballPosExtended:To2D().y, 1, Draw.Color(255, 138,43,226))
					Draw.Circle(ballPosExtended, ball.object.boundingRadius, 2, Draw.Color(120, 138,43,226))
				end
			end
		end
	end
end

--[[                                                                                                   
                                                                                                        
   SSSSSSSSSSSSSSS                                                              iiii                    
 SS:::::::::::::::S                                                            i::::i                   
S:::::SSSSSS::::::S                                                             iiii                    
S:::::S     SSSSSSS                                                                                     
S:::::S            wwwwwww           wwwww           wwwwwww  aaaaaaaaaaaaa   iiiiiii nnnn  nnnnnnnn    
S:::::S             w:::::w         w:::::w         w:::::w   a::::::::::::a  i:::::i n:::nn::::::::nn  
 S::::SSSS           w:::::w       w:::::::w       w:::::w    aaaaaaaaa:::::a  i::::i n::::::::::::::nn 
  SS::::::SSSSS       w:::::w     w:::::::::w     w:::::w              a::::a  i::::i nn:::::::::::::::n
    SSS::::::::SS      w:::::w   w:::::w:::::w   w:::::w        aaaaaaa:::::a  i::::i   n:::::nnnn:::::n
       SSSSSS::::S      w:::::w w:::::w w:::::w w:::::w       aa::::::::::::a  i::::i   n::::n    n::::n
            S:::::S      w:::::w:::::w   w:::::w:::::w       a::::aaaa::::::a  i::::i   n::::n    n::::n
            S:::::S       w:::::::::w     w:::::::::w       a::::a    a:::::a  i::::i   n::::n    n::::n
SSSSSSS     S:::::S        w:::::::w       w:::::::w        a::::a    a:::::a i::::::i  n::::n    n::::n
S::::::SSSSSS:::::S         w:::::w         w:::::w         a:::::aaaa::::::a i::::::i  n::::n    n::::n
S:::::::::::::::SS           w:::w           w:::w           a::::::::::aa:::ai::::::i  n::::n    n::::n
 SSSSSSSSSSSSSSS              www             www             aaaaaaaaaa  aaaaiiiiiiii  nnnnnn    nnnnnn

]]                                                                                             

class "Swain"

function Swain:__init()

	if charName ~= "Swain" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Swain:LoadSpells()

	Q = { range = 700, 	delay = 0.25 } -- no collision, sux higher lvl 
	W = { range = 3500,	width = 125,	speed = _huge,	delay = 0.25 }
	E = { range = 950,		width = 65,		speed = 775,	delay = 0.33, collision = Collision:SetSpell(950, 775, 0.33, 65, false) }
	R = { range = myHero:GetSpellData(_R).range }
end

function Swain:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "disableAA", name = "Use Smart AA switch", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Swain:OnTick()

	if Menu.Combo.disableAA:Value() and Utils:HasBuff(myHero, "swainr") and self:AllowAttacks() == false then
		Orb:DisableAttacks(true)
	else
		Orb:DisableAttacks(false)
	end

	if myHero.attackData.state ~= STATE_WINDUP and Orb.canAttack then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, E.range) > 0 then
			for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, E.range) do
				local enemy = Utils:GetEnemyHeroesInRange(myHero.pos, E.range)[i]
				if enemy then
					if Utils:IsValid(enemy, myHero.pos, E.range) and Utils:IsImmobile(enemy) and not myHero.attackData.state == STATE_WINDUP then
						Control.Attack(enemy)
						break
					end
				end
			end
		end
	end

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Swain:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		Utils:CastSpell(HK_Q, unit.pos, Q.range)
	end
end

function Swain:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) and Utils:IsImmobile(unit) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		if t and aimPos and t.networkID == unit.networkID then
			if Utils:GetDistance(aimPos, myHero.pos) < W.range then
				if aimPos:To2D().onScreen  then
					Utils:CastSpell(HK_W, aimPos, W.range)
				else
					Utils:CastSpellMM(HK_W, aimPos, W.range)
				end
			end
		end
	end
end

function Swain:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, E.speed, E.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
					local col, units = E.collision:__GetCollision(myHero.pos:Extended(nPred, Utils:GetDistance(myHero.pos, nPred) - 50), myHero.pos:Extended(nPred, E.range), 5)
					if not col then
						Utils:CastSpell(HK_E, nPred, E.range)
						return
					end
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
					local col, units = E.collision:__GetCollision(myHero.pos:Extended(aimPos, Utils:GetDistance(myHero.pos, aimPos) - 50), myHero.pos:Extended(aimPos, E.range), 5)
					if not col then
						Utils:CastSpell(HK_E, aimPos, E.range)
						return
					end
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) <= E.range then
						local col, units = E.collision:__GetCollision(myHero.pos:Extended(aimPos2, Utils:GetDistance(myHero.pos, aimPos2) - 50), myHero.pos:Extended(aimPos2, E.range), 5)
						if not col then
							Utils:CastSpell(HK_E, aimPos2, E.range)
							return
						end
					end
				end
			end
		end
	end
end

function Swain:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Swain:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Swain:UltCharged()
	
	if not Utils:IsReady(_R) then return end
	for i = 1, Game.ParticleCount() do
		local particle = Game.Particle(i)
		if particle and Utils:GetDistance(myHero.pos, particle.pos) <= 200 and _find(particle.name:lower(), "r_full") then
			return true
		end
	end
	return false
end

function Swain:AllowAttacks()

	for i = 1, myHero.buffCount do
		local buff = myHero:GetBuff(i)
		if buff and buff.count > 0 and (_find(buff.name:lower(), "itemfrozenfist") or _find(buff.name:lower(), "sheen") 
			or _find(buff.name:lower(), "lichbane")) then
			return true
		end
	end
	return false
end

function Swain:OnDraw()

	if self:UltCharged() then
		Draw.Circle(myHero.pos, R.range, 8, Draw.Color(180, 255, 0, 0))
	end
end

--[[

                                                                                                                            
                                                                                                  dddddddd                  
TTTTTTTTTTTTTTTTTTTTTTT                                                                           d::::::d                  
T:::::::::::::::::::::T                                                                           d::::::d                  
T:::::::::::::::::::::T                                                                           d::::::d                  
T:::::TT:::::::TT:::::T                                                                           d:::::d                   
TTTTTT  T:::::T  TTTTTTrrrrr   rrrrrrrrr   yyyyyyy           yyyyyyynnnn  nnnnnnnn        ddddddddd:::::d   aaaaaaaaaaaaa   
        T:::::T        r::::rrr:::::::::r   y:::::y         y:::::y n:::nn::::::::nn    dd::::::::::::::d   a::::::::::::a  
        T:::::T        r:::::::::::::::::r   y:::::y       y:::::y  n::::::::::::::nn  d::::::::::::::::d   aaaaaaaaa:::::a 
        T:::::T        rr::::::rrrrr::::::r   y:::::y     y:::::y   nn:::::::::::::::nd:::::::ddddd:::::d            a::::a 
        T:::::T         r:::::r     r:::::r    y:::::y   y:::::y      n:::::nnnn:::::nd::::::d    d:::::d     aaaaaaa:::::a 
        T:::::T         r:::::r     rrrrrrr     y:::::y y:::::y       n::::n    n::::nd:::::d     d:::::d   aa::::::::::::a 
        T:::::T         r:::::r                  y:::::y:::::y        n::::n    n::::nd:::::d     d:::::d  a::::aaaa::::::a 
        T:::::T         r:::::r                   y:::::::::y         n::::n    n::::nd:::::d     d:::::d a::::a    a:::::a 
      TT:::::::TT       r:::::r                    y:::::::y          n::::n    n::::nd::::::ddddd::::::dda::::a    a:::::a 
      T:::::::::T       r:::::r                     y:::::y           n::::n    n::::n d:::::::::::::::::da:::::aaaa::::::a 
      T:::::::::T       r:::::r                    y:::::y            n::::n    n::::n  d:::::::::ddd::::d a::::::::::aa:::a
      TTTTTTTTTTT       rrrrrrr                   y:::::y             nnnnnn    nnnnnn   ddddddddd   ddddd  aaaaaaaaaa  aaaa
                                                 y:::::y                                                                    
                                                y:::::y                                                                     
                                               y:::::y                                                                      
                                              y:::::y                                                                       
                                             yyyyyyy                                                                        

--]]

class "Tryndamere"

function Tryndamere:__init()

	if charName ~= "Tryndamere" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
end

function Tryndamere:LoadSpells()

	Q = { range = 600,	delay = 0.25 }
	W = { range = 850 }
	E = { range = 660, 	delay = 0.1,	width =  225}
	R = { range = 500 }
end

function Tryndamere:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "qHp", name = "HP % to use Q", value = 50, min = 0, max = 100, step = 1})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})
		Menu.Combo:MenuElement({id = "useRhp", name = "Use R at x %HP", value = 15, min = 0, max = 100, step = 1})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Tryndamere:OnTick()

	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q) and Utils:GetHpPercent(myHero) <= Menu.Combo.qHp:Value() and myHero.mana >= 50
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R) and Utils:GetHpPercent(myHero) <= Menu.Combo.useRhp:Value()

	if qReady then
		self:CastQcombo()
	end

	if rReady then
		self:CastRcombo()
	end

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Tryndamere:CastQcombo()

	for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, 2000) do
		local current = Utils:GetEnemyHeroesInRange(myHero.pos, 2000)[i]
		if current then
			if current.activeSpell and current.activeSpell.valid and
				(current.activeSpell.target == myHero.handle or 
					Utils:GetDistance(current.activeSpell.placementPos, myHero.pos) <= myHero.boundingRadius * 2 + current.activeSpell.width) then
				local startPos = current.activeSpell.startPos
				local placementPos = current.activeSpell.placementPos
				local width = 0
				if current.activeSpell.width > 0 then
					width = current.activeSpell.width
				else
					width = 100
				end
				local distance = Utils:GetDistance(myHero.pos, placementPos)											
				if current.activeSpell.target == myHero.handle then
					Control.CastSpell(HK_Q)
					return
				else
					if distance <= width * 2 + myHero.boundingRadius then
						Control.CastSpell(HK_Q)
						break						
					end
				end
			end
		end
	end
end

function Tryndamere:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		if unit.pathing.hasMovePath and Utils:GetDistance(unit.pathing.endPos, myHero.pos) > W.range then
			Control.CastSpell(HK_W)
		end
	end
end

function Tryndamere:CastEcombo()

	if #Utils:GetEnemyHeroesInRange(myHero.pos, E.range) > 0 then
		for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, E.range) do
			local enemy = Utils:GetEnemyHeroesInRange(myHero.pos, E.range)[i]
			if enemy then
				if self:GetComboDmg(enemy) > enemy.health or Utils:GetDistance(enemy.pos, myHero.pos) > myHero.range then
					Utils:CastSpell(HK_E, enemy.pos, E.range)
					break
				end
			end
		end
	end
end

function Tryndamere:CastRcombo()

	for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, 2000) do
		local current = Utils:GetEnemyHeroesInRange(myHero.pos, 2000)[i]
		if current then
			if current.activeSpell and current.activeSpell.valid and
				(current.activeSpell.target == myHero.handle or 
					Utils:GetDistance(current.activeSpell.placementPos, myHero.pos) <= myHero.boundingRadius * 2 + current.activeSpell.width) then
				local startPos = current.activeSpell.startPos
				local placementPos = current.activeSpell.placementPos
				local width = 0
				if current.activeSpell.width > 0 then
					width = current.activeSpell.width
				else
					width = 100
				end
				local distance = Utils:GetDistance(myHero.pos, placementPos)											
				if current.activeSpell.target == myHero.handle then
					Control.CastSpell(HK_R)
					return
				else
					if distance <= width * 2 + myHero.boundingRadius then
						Control.CastSpell(HK_R)
						break
					end
				end
			end
		end
	end
end

function Tryndamere:Combo()

	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Tryndamere:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Tryndamere:GetComboDmg(unit)

	local totalDmg = 0
	if unit then
		local eLvl = myHero:GetSpellData(_E).level
		local rLvl = myHero:GetSpellData(_R).level
		if eLvl > 0 then
			totalDmg = totalDmg + self:GetEdmg(unit)
		end
		if rLvl > 0 and Utils:IsReady(_R) then
			totalDmg = totalDmg + (myHero.totalDamage * 1.5) * 5
		end
	end
	return totalDmg
end

function Tryndamere:GetEdmg(unit)

	local totalDmg = 0
	if unit then
		local eLvl = myHero:GetSpellData(_E).level
		if eLvl > 0 then
			local baseDmg = ({ 80, 110, 140, 170, 200 })[eLvl]
			local tDmg = myHero.totalDamage
			local bonusAd = 1.3 * myHero.bonusDamage
			local ap = myHero.ap
			totalDmg = baseDmg + bonusAd + ap + tDmg
		end
	end
	return Utils:CalculateMagicalDamage(unit, totalDmg)
end

--[[                                                                                                   
                                                                                                      
DDDDDDDDDDDDD                                                iiii                                     
D::::::::::::DDD                                            i::::i                                    
D:::::::::::::::DD                                           iiii                                     
DDD:::::DDDDD:::::D                                                                                   
  D:::::D    D:::::D   aaaaaaaaaaaaa   rrrrr   rrrrrrrrr   iiiiiii uuuuuu    uuuuuu      ssssssssss   
  D:::::D     D:::::D  a::::::::::::a  r::::rrr:::::::::r  i:::::i u::::u    u::::u    ss::::::::::s  
  D:::::D     D:::::D  aaaaaaaaa:::::a r:::::::::::::::::r  i::::i u::::u    u::::u  ss:::::::::::::s 
  D:::::D     D:::::D           a::::a rr::::::rrrrr::::::r i::::i u::::u    u::::u  s::::::ssss:::::s
  D:::::D     D:::::D    aaaaaaa:::::a  r:::::r     r:::::r i::::i u::::u    u::::u   s:::::s  ssssss 
  D:::::D     D:::::D  aa::::::::::::a  r:::::r     rrrrrrr i::::i u::::u    u::::u     s::::::s      
  D:::::D     D:::::D a::::aaaa::::::a  r:::::r             i::::i u::::u    u::::u        s::::::s   
  D:::::D    D:::::D a::::a    a:::::a  r:::::r             i::::i u:::::uuuu:::::u  ssssss   s:::::s 
DDD:::::DDDDD:::::D  a::::a    a:::::a  r:::::r            i::::::iu:::::::::::::::uus:::::ssss::::::s
D:::::::::::::::DD   a:::::aaaa::::::a  r:::::r            i::::::i u:::::::::::::::us::::::::::::::s 
D::::::::::::DDD      a::::::::::aa:::a r:::::r            i::::::i  uu::::::::uu:::u s:::::::::::ss  
DDDDDDDDDDDDD          aaaaaaaaaa  aaaa rrrrrrr            iiiiiiii    uuuuuuuu  uuuu  sssssssssss    
                                                                                                 
--]]

class "Darius"

function Darius:__init()

	if charName ~= "Darius" then return end

	self.lastMove = GetTickCount()
	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)

	if _G.SDK then
        _G.SDK.Orbwalker:OnPostAttack(function() self:OnPostAttack() end)
    elseif _G.EOW then
        EOW:AddCallback(EOW.AfterAttack, function() self:OnPostAttack() end)
    elseif _G.GOS then
        GOS:OnAttackComplete(function() self:OnPostAttack() end)
    end
end

function Darius:LoadSpells()

	Q = { range = 450,	outerRadius = 425, innerRadius = 205, delay = 0.25 }
	W = { range = 200 }
	E = { range = 535 }
	R = { range = myHero:GetSpellData(_R).range }
end

function Darius:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Darius:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = nil

	if Utils:IsReady(_R) then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, R.range) > 0 then
			for _, v in pairs(Utils:GetEnemyHeroesInRange(myHero.pos, R.range)) do
				local enemy = v

				if enemy then
					if enemy.health < self:GetRdmg(enemy) then
						self.rTarget = enemy
					end
				end
			end
		end
	end

	if #Utils:GetEnemyHeroesInRange(myHero.pos, Q.range) == 0 then
		Orb:DisableMovement(false)
	end

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Darius:CastQcombo(unit)

	if unit and Utils:IsValid(unit, myHero.pos, Q.outerRadius) then
		Control.CastSpell(HK_Q)
	end
end

function Darius:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, myHero.range + 50) then
		if Utils:GetDistance(unit.pathing.endPos, myHero.pos) > myHero.range + 50 then
			Control.CastSpell(HK_W)
		end
	end
end

function Darius:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		Utils:CastSpell(HK_E, unit.pos, E.range)
	end
end

function Darius:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) then
		Control.CastSpell(HK_R, unit)
	end
end

function Darius:OnPostAttack()

	if self.wTarget and self.wTarget.handle == myHero.attackData.target then
		if Utils:IsReady(_W) and Menu.Combo.useW:Value() then
			Control.CastSpell(HK_W)
		end
	end
end

function Darius:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if self.qTarget then
		if Utils:HasBuff(myHero, "dariusqcast") then
			Orb:DisableMovement(true)
			if self.lastMove + 120 < GetTickCount() then
				if Utils:GetDistance(self.qTarget.pos, myHero.pos) > 375 then
					Control.Move(myHero.pos:Extended(self.qTarget.pos, Utils:GetDistance(self.qTarget.pos, myHero.pos) + 500))
					self.lastMove = GetTickCount()
				end
				if Utils:GetDistance(self.qTarget.pos, myHero.pos) < 250 then
					Control.Move(self.qTarget.pos:Extended(myHero.pos, Utils:GetDistance(self.qTarget.pos, myHero.pos) + 500))
					self.lastMove = GetTickCount()
				end
			end
		else
			Orb:DisableMovement(false)
		end
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Darius:GetHemoCount(unit)
	
	if unit then
		for i = 1, unit.buffCount do
			local buff = unit:GetBuff(i)
			if buff and buff.count > 0 and buff.name:lower() == "dariushemo" then 
				return buff.count
			end
		end
	end
	return 0
end

function Darius:GetRdmg(unit)

	local totalDmg = 0
	if unit then
		local rLvl = myHero:GetSpellData(_R).level
		if rLvl > 0 then
			local count = self:GetHemoCount(unit)
			local stackDmg = ({ 20, 40, 60 })[rLvl]
			local baseDmg = ({ 100, 200, 300 })[rLvl]
			local ad = 0.75 * myHero.bonusDamage
			local ad2 = 0.15 * myHero.bonusDamage
			totalDmg = totalDmg + baseDmg + ad + (count * (stackDmg + ad2))
		end
	end
	return totalDmg
end

function Darius:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

--[[                                                                   
                                                                   
          JJJJJJJJJJJhhhhhhh               iiii                    
          J:::::::::Jh:::::h              i::::i                   
          J:::::::::Jh:::::h               iiii                    
          JJ:::::::JJh:::::h                                       
            J:::::J   h::::h hhhhh       iiiiiii nnnn  nnnnnnnn    
            J:::::J   h::::hh:::::hhh    i:::::i n:::nn::::::::nn  
            J:::::J   h::::::::::::::hh   i::::i n::::::::::::::nn 
            J:::::j   h:::::::hhh::::::h  i::::i nn:::::::::::::::n
            J:::::J   h::::::h   h::::::h i::::i   n:::::nnnn:::::n
JJJJJJJ     J:::::J   h:::::h     h:::::h i::::i   n::::n    n::::n
J:::::J     J:::::J   h:::::h     h:::::h i::::i   n::::n    n::::n
J::::::J   J::::::J   h:::::h     h:::::h i::::i   n::::n    n::::n
J:::::::JJJ:::::::J   h:::::h     h:::::hi::::::i  n::::n    n::::n
 JJ:::::::::::::JJ    h:::::h     h:::::hi::::::i  n::::n    n::::n
   JJ:::::::::JJ      h:::::h     h:::::hi::::::i  n::::n    n::::n
     JJJJJJJJJ        hhhhhhh     hhhhhhhiiiiiiii  nnnnnn    nnnnnn
                                                             
--]]

class "Jhin"

function Jhin:__init()

	if charName ~= "Jhin" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Jhin:LoadSpells()

	attackRange = 550
	Q = { range = 550, 		bounceRadius = 400 }
	W = { range = 3000, 	speed = 5000, 		width = 40,		delay = 0.75 }
	E = { range = 750, 		speed = _huge,		width = 120,	delay = 0.25 }
	R = { range = 3500, 	speed = 5000, 		width = 80,		delay = 0.6, 	collision = Collision:SetSpell(3500, 5000, 0, 80, true) }
end

function Jhin:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useQtoggle", name = "Passive Logic Toggle", key = string.byte("T"), toggle = true })
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})
		Menu.Combo:MenuElement({id = "useKs", name = "KS everything!", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end
	
function Jhin:OnTick()

	self.qTarget = Utils:GetTarget(Q.range + Q.bounceRadius)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range + E.width)
	self.rTarget = Utils:GetTarget(R.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Jhin:IsUlting()
	return Utils:IsParticleInRange("jhin_base_r_viewcone", myHero.pos, 200)
end

function Jhin:CastQcombo(unit)

	if myHero.attackData.state ~= STATE_WINDUP or not Orb.canAttack then
		if Utils:IsValid(unit, myHero.pos, Q.range) then
			Utils:CastSpell(HK_Q, unit.pos, Q.range)
		end
	end
end

function Jhin:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) and (Utils:HasBuff(unit, "jhinespotteddebuff") or self:GetWdmg(unit) > unit.health) and 
			(myHero.attackData.state ~= STATE_WINDUP or not Orb.canAttack) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, nPred, W.range)
				else
					Utils:CastSpell(HK_W, myHero.pos:Extended(nPred, 500), W.range)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < W.range then
					Utils:CastSpell(HK_W, aimPos, W.range)
				else
					Utils:CastSpell(HK_W, myHero.pos:Extended(aimPos, 500), W.range)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy or (hitChance >= 1 and unit.health < self:GetWdmg(unit)) then
					if Utils:GetDistance(aimPos2, myHero.pos) < W.range then
						if aimPos2:To2D().onScreen then
							Utils:CastSpell(HK_W, aimPos2, W.range)
						else
							Utils:CastSpell(HK_W, myHero.pos:Extended(aimPos2, 500), W.range)
						end
					end
				end
			end
		end
	end
end

function Jhin:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) and myHero.attackData.state ~= STATE_WINDUP  then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, E.speed, E.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
					Utils:CastSpell(HK_E, nPred, E.range)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
					Utils:CastSpell(HK_E, aimPos, E.range)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy or (hitChance >= 1 and unit.health < self:GetWdmg(unit)) then
					if Utils:GetDistance(aimPos2, myHero.pos) < E.range then
						if aimPos2:To2D().onScreen then
							Utils:CastSpell(HK_E, aimPos2, E.range)
						else
							Utils:CastSpell(HK_E, myHero.pos:Extended(aimPos2, 500), E.range)
						end
					end
				end
			end
		end
	end
end

function Jhin:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, R.range, R.delay, R.speed, R.width, .25, false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, R.range, R.delay, R.speed, R.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, R.speed, R.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if Utils:GetDistance(nPred, myHero.pos) < R.range then
					local col, units = R.collision:__GetHeroCollision(myHero.pos, nPred, 3, unit)
					if not col then
						if nPred:To2D().onScreen then
							Utils:CastSpell(HK_R, nPred, R.range)
							return
						else
							Utils:CastSpell(HK_R, myHero.pos:Extended(nPred, 500), R.range)
							return
						end
					end
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if Utils:GetDistance(aimPos, myHero.pos) < R.range then
					local col, units = R.collision:__GetHeroCollision(myHero.pos, aimPos, 3, unit)
					if not col then
						if aimPos:To2D().onScreen then
							Utils:CastSpell(HK_R, aimPos, R.range)
							return
						else
							Utils:CastSpell(HK_R, myHero.pos:Extended(aimPos, 500), R.range)
							return
						end
					end
				end
			end
			if hitChance and aimPos2 then
				if hitChance and aimPos2 then
					if hitChance >= 2 then
						if Utils:GetDistance(aimPos2, myHero.pos) < R.range then
							local col, units = R.collision:__GetHeroCollision(myHero.pos, aimPos2, 3, unit)
							if not col then
								if aimPos2:To2D().onScreen then
									Utils:CastSpell(HK_R, aimPos2, R.range)
									return
								else
									Utils:CastSpell(HK_R, myHero.pos:Extended(aimPos2, 500), R.range)
								end
							end
						end
					end
				end
			end
		end
	end
end

function Jhin:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if not self:IsUlting() then
		if qReady and self.qTarget then
			self:CastQcombo(self.qTarget)
		end

		if wReady and self.wTarget then
			self:CastWcombo(self.wTarget)
		end

		if eReady and self.eTarget then
			self:CastEcombo(self.eTarget)
		end
	else
		if self.rTarget then
			self:CastRcombo(self.rTarget)
		end
	end
end

function Jhin:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if not self:IsUlting() then
		if qReady and self.qTarget then
			self:CastQcombo(self.qTarget)
		end

		if wReady and self.wTarget then
			self:CastWcombo(self.wTarget)
		end

		if eReady and self.eTarget then
			self:CastEcombo(self.eTarget)
		end
	end
end

function Jhin:GetWdmg(unit)

	local totalDmg = 0
	if unit then
		local wLvl = myHero:GetSpellData(_W).level
		if wLvl > 0 then
			local baseDmg = ({ 50, 85, 120, 155, 190 })[wLvl]
			local bonusAd = myHero.totalDamage * .5

			totalDmg = baseDmg + bonusAd
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Jhin:OnDraw()

	if self:IsUlting() then
		Orb:DisableMovement(true)
	else
		Orb:DisableMovement(false)
	end
end

--[[                                                                       
                                                                             
   SSSSSSSSSSSSSSS   iiii                             iiii                      
 SS:::::::::::::::S i::::i                           i::::i                     
S:::::SSSSSS::::::S  iiii                             iiii                      
S:::::S     SSSSSSS                                                             
S:::::S            iiiiiii vvvvvvv           vvvvvvviiiiiii rrrrr   rrrrrrrrr   
S:::::S            i:::::i  v:::::v         v:::::v i:::::i r::::rrr:::::::::r  
 S::::SSSS          i::::i   v:::::v       v:::::v   i::::i r:::::::::::::::::r 
  SS::::::SSSSS     i::::i    v:::::v     v:::::v    i::::i rr::::::rrrrr::::::r
    SSS::::::::SS   i::::i     v:::::v   v:::::v     i::::i  r:::::r     r:::::r
       SSSSSS::::S  i::::i      v:::::v v:::::v      i::::i  r:::::r     rrrrrrr
            S:::::S i::::i       v:::::v:::::v       i::::i  r:::::r            
            S:::::S i::::i        v:::::::::v        i::::i  r:::::r            
SSSSSSS     S:::::Si::::::i        v:::::::v        i::::::i r:::::r            
S::::::SSSSSS:::::Si::::::i         v:::::v         i::::::i r:::::r            
S:::::::::::::::SS i::::::i          v:::v          i::::::i r:::::r            
 SSSSSSSSSSSSSSS   iiiiiiii           vvv           iiiiiiii rrrrrrr            
                                                                                                                                                            
--]]

class "Sivir"

function Sivir:__init()

	if charName ~= "Sivir" then return end

	self:LoadSpells()
	self:LoadMenu()

	self.SpellsLoaded = false

	Callback.Add("Tick", function() self:OnTick() end)
	if _G.SDK then
        _G.SDK.Orbwalker:OnPostAttack(function() self:OnPostAttack() end)
    elseif _G.EOW then
        EOW:AddCallback(EOW.AfterAttack, function() self:OnPostAttack() end)
    elseif _G.GOS then
        GOS:OnAttackComplete(function() self:OnPostAttack() end)
    end
end

function Sivir:LoadSpells()

	Q = { range = 1250,	delay = 0.25, speed = 1350, width = 90 }
	W = { range = myHero.range }
	E = { }
	R = { }
end

function Sivir:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true })
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true })
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true })
		Menu.Combo:MenuElement({id = "eSettings", name = "E settings", type = MENU})
		Menu.Combo.eSettings:MenuElement({id = "spells", name = "Spells to block", type = MENU})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})

	--- Misc ---
		Menu.Misc:MenuElement({id = "drawSpells", name = "Draw blocked spells", value = true})
end

function Sivir:LoadBlockSpells()

	for i = 1, LocalGameHeroCount(i) do
		local t = LocalGameHero(i)
		if t and t.isEnemy then		
			for slot = 0, 3 do
				local enemy = t
				local spellName = enemy:GetSpellData(slot).name
				if slot == 0 then
					Menu.Combo.eSettings.spells:MenuElement({ id = spellName, name = enemy.charName.."- Q", value = true })
				end
				if slot == 1 then
					Menu.Combo.eSettings.spells:MenuElement({ id = spellName, name = enemy.charName.."- W", value = true })
				end
				if slot == 2 then
					Menu.Combo.eSettings.spells:MenuElement({ id = spellName, name = enemy.charName.."- E", value = true })
				end
				if slot == 3 then
					Menu.Combo.eSettings.spells:MenuElement({ id = spellName, name = enemy.charName.."- R", value = true })
				end			
			end
		end
	end
end

function Sivir:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	local currentTime = LocalGameTimer()
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	
	if not self.SpellsLoaded and LocalGameTimer() > 10 then
		self:LoadBlockSpells()
		self.SpellsLoaded = true
	end

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end

	if eReady and self.SpellsLoaded == true then
		for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, 2000) do
			local current = Utils:GetEnemyHeroesInRange(myHero.pos, 2000)[i]
			if current then
				if current.activeSpell and current.activeSpell.valid and
					(current.activeSpell.target == myHero.handle or 
						Utils:GetDistance(current.activeSpell.placementPos, myHero.pos) <= myHero.boundingRadius * 2 + current.activeSpell.width) and not 
						_find(current.activeSpell.name:lower(), "attack") then
					for j = 0, 3 do
						local spell = current:GetSpellData(j)
						--print("ActiveName: "..current.activeSpell.name.. " SpellName: "..spell.name)
						if Menu.Combo.eSettings.spells[spell.name] and Menu.Combo.eSettings.spells[spell.name]:Value() and spell.name == current.activeSpell.name then
							local startPos = current.activeSpell.startPos
							local placementPos = current.activeSpell.placementPos
							local width = 0
							if current.activeSpell.width > 0 then
								width = current.activeSpell.width
							else
								width = 100
							end
							local distance = Utils:GetDistance(myHero.pos, placementPos)											
							if current.activeSpell.target == myHero.handle then
								if Menu.Misc.drawSpells:Value() then
									print("Blocking(Targeted): " ..spell.name)
								end
								Control.CastSpell(HK_E)
								return
							else
								if distance <= width * 2 + myHero.boundingRadius then
									if Menu.Misc.drawSpells:Value() then
										print("Blocking(Skillshot): " ..spell.name)
									end
									Control.CastSpell(HK_E)
									break
								end
							end							
						end
					end
				end
			end
		end
	end
end

function Sivir:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		local range = myHero.range + unit.boundingRadius
		if (Utils:GetDistance(unit.pos, myHero.pos) <= range and myHero.attackData.state ~= STATE_WINDUP) or 
			Utils:GetDistance(unit.pos, myHero.pos) > range then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.range then
						Utils:CastSpell(HK_Q, nPred, Q.range)
					end					
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < Q.range then
						Utils:CastSpell(HK_Q, aimPos, Q.range)
					end
				end
				if hitChance and aimPos2 then
					if hitChance >= minAccuracy or (hitChance >= 1 and unit.health < self:GetQdmg(unit)) then
						if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < Q.range then
							Utils:CastSpell(HK_Q, aimPos2, Q.range)
						end
					end
				end
			end
		end
	end
end

function Sivir:OnPostAttack()

	if Orb.combo or Orb.harass then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, myHero.range + 100) > 0 then
			if Utils:IsReady(_W) and Menu.Combo.useW:Value() then
				Control.CastSpell(HK_W)
			end
		end
	end
end

function Sivir:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end
end

function Sivir:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end
end

function Sivir:GetQdmg(unit)

	local totalDmg = 0
	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 35, 55, 75, 95, 115 })[qLvl]
			local bonusAdMod = ({ .7, .8, .9, 1, 1.1 })[qLvl]
			local bonusAd = myHero.totalDamage * bonusAdMod

			totalDmg = (baseDmg + bonusAd) * 1.85
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

--[[                                                                                           
                                                                                              
NNNNNNNN        NNNNNNNN                                                                      
N:::::::N       N::::::N                                                                      
N::::::::N      N::::::N                                                                      
N:::::::::N     N::::::N                                                                      
N::::::::::N    N::::::N  aaaaaaaaaaaaa       ssssssssss   uuuuuu    uuuuuu      ssssssssss   
N:::::::::::N   N::::::N  a::::::::::::a    ss::::::::::s  u::::u    u::::u    ss::::::::::s  
N:::::::N::::N  N::::::N  aaaaaaaaa:::::a ss:::::::::::::s u::::u    u::::u  ss:::::::::::::s 
N::::::N N::::N N::::::N           a::::a s::::::ssss:::::su::::u    u::::u  s::::::ssss:::::s
N::::::N  N::::N:::::::N    aaaaaaa:::::a  s:::::s  ssssss u::::u    u::::u   s:::::s  ssssss 
N::::::N   N:::::::::::N  aa::::::::::::a    s::::::s      u::::u    u::::u     s::::::s      
N::::::N    N::::::::::N a::::aaaa::::::a       s::::::s   u::::u    u::::u        s::::::s   
N::::::N     N:::::::::Na::::a    a:::::a ssssss   s:::::s u:::::uuuu:::::u  ssssss   s:::::s 
N::::::N      N::::::::Na::::a    a:::::a s:::::ssss::::::su:::::::::::::::uus:::::ssss::::::s
N::::::N       N:::::::Na:::::aaaa::::::a s::::::::::::::s  u:::::::::::::::us::::::::::::::s 
N::::::N        N::::::N a::::::::::aa:::a s:::::::::::ss    uu::::::::uu:::u s:::::::::::ss  
NNNNNNNN         NNNNNNN  aaaaaaaaaa  aaaa  sssssssssss        uuuuuuuu  uuuu  sssssssssss    

--]]

class "Nasus"

function Nasus:__init()

	if charName ~= "Nasus" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.lastObjectLoop = GetTickCount()

	Callback.Add("Tick", function() self:OnTick() end)
	if _G.SDK then
        _G.SDK.Orbwalker:OnPostAttack(function() self:OnPostAttack() end)
    elseif _G.EOW then
        EOW:AddCallback(EOW.AfterAttack, function() self:OnPostAttack() end)
    elseif _G.GOS then
        GOS:OnAttackComplete(function() self:OnPostAttack() end)
    end
    Callback.Add("Draw", function() self:OnDraw() end)
end

function Nasus:LoadSpells()

	attackRange = myHero.range + myHero.boundingRadius
	Q = { range = attackRange + 25 }
	W = { range = 600 }
	E = { range = 650,	width = 400}
	R = { }
	Stacks = { count = 0, last = GetTickCount()}
end

function Nasus:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Misc ---
		Menu.Misc:MenuElement({id = "drawLasthit", name = "Draw Lasthit minions", value = true})
end

function Nasus:OnTick()

	self.qTarget = Utils:GetTarget(Q.range + 100)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range + E.width)

	if Stacks.last + 1000 < GetTickCount() then
		self:GetStacks()
	end
	self:UpdateRange()
	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	elseif Orb.laneClear or Orb.jungleClear or Orb.lastHit then
		self:Clear()
	end
end

function Nasus:OnPostAttack()

	if Orb.combo or Orb.harass then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, Q.range + myHero.boundingRadius * 2) > 0 then
			if Utils:IsReady(_Q) and Menu.Combo.useQ:Value() then
				Control.CastSpell(HK_Q)
			end
		end
	end
end

function Nasus:CastQcombo(unit)

	if unit and Utils:IsValid(unit, myHero.pos, Q.range + unit.boundingRadius + myHero.boundingRadius) and myHero.attackData.state ~= STATE_WINDUP then
		Control.CastSpell(HK_Q)
		Control.Attack(unit)
	end
end

function Nasus:CastWcombo(unit)

	if unit and Utils:IsValid(unit, myHero.pos, W.range) and myHero.attackData.state ~= STATE_WINDUP then
		Utils:CastSpell(HK_W, unit.pos, W.range)
	end
end

function Nasus:CastEcombo(unit)

	if unit and Utils:IsValid(unit, myHero.pos, E.range + E.width) and myHero.attackData.state ~= STATE_WINDUP then
		local pred = Utils:GetPred(unit, unit.ms)
		if pred then
			Utils:CastSpell(HK_E, pred, E.range)
		end
	end
end

function Nasus:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Nasus:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Nasus:Clear()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)

	if qReady then
		for i = 1, LocalGameMinionCount() do
			local object = LocalGameMinion(i)
			if object and not object.isAlly then
				if Utils:IsValid(object, myHero.pos, Q.range + myHero.boundingRadius + object.boundingRadius) then
					if self:GetQdmg(object) > object.health then
						Control.CastSpell(HK_Q)
						Control.Attack(object)
						break
					end
				end
			end
		end
	end
end

function Nasus:GetStacks()
	
	if Utils:HasBuff(myHero, "nasusqstacks") then
		for i = 1, myHero.buffCount do
			local buff = myHero:GetBuff(i)
			if buff and buff.name:lower() == "nasusqstacks" then
				Stacks.count = buff.stacks
				break
			end
		end
	end
	Stacks.last = GetTickCount()
end

function Nasus:GetQdmg(unit)
	
	local totalDmg = 0
	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 30, 50, 70, 90, 110  })[qLvl]
			local stacks = Stacks.count

			totalDmg = myHero.totalDamage + (baseDmg + stacks)
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Nasus:UpdateRange()
		
	local range = attackRange + 175

	if Utils:HasBuff(myHero, "nasusr") then
		Q.range = range
	else
		Q.range = attackRange + 25
	end
end

function Nasus:OnDraw()

	if Menu.Misc.drawLasthit:Value() then
		for i = 1, LocalGameMinionCount() do
			local object = LocalGameMinion(i)
			if object and not object.isAlly then
				if Utils:IsValid(object, myHero.pos, 1000) then
					if self:GetQdmg(object) > object.health then
						Draw.Circle(object.pos, 50, 3, Draw.Color(180, 0, 255, 0))
					end
				end
			end
		end
	end
end

--[[                                                                                                                                                   
                                                                                                                                                    
MMMMMMMM               MMMMMMMM                  lllllll                     hhhhhhh               iiii           tttt                              
M:::::::M             M:::::::M                  l:::::l                     h:::::h              i::::i       ttt:::t                              
M::::::::M           M::::::::M                  l:::::l                     h:::::h               iiii        t:::::t                              
M:::::::::M         M:::::::::M                  l:::::l                     h:::::h                           t:::::t                              
M::::::::::M       M::::::::::M  aaaaaaaaaaaaa    l::::l ppppp   ppppppppp    h::::h hhhhh       iiiiiii ttttttt:::::ttttttt        eeeeeeeeeeee    
M:::::::::::M     M:::::::::::M  a::::::::::::a   l::::l p::::ppp:::::::::p   h::::hh:::::hhh    i:::::i t:::::::::::::::::t      ee::::::::::::ee  
M:::::::M::::M   M::::M:::::::M  aaaaaaaaa:::::a  l::::l p:::::::::::::::::p  h::::::::::::::hh   i::::i t:::::::::::::::::t     e::::::eeeee:::::ee
M::::::M M::::M M::::M M::::::M           a::::a  l::::l pp::::::ppppp::::::p h:::::::hhh::::::h  i::::i tttttt:::::::tttttt    e::::::e     e:::::e
M::::::M  M::::M::::M  M::::::M    aaaaaaa:::::a  l::::l  p:::::p     p:::::p h::::::h   h::::::h i::::i       t:::::t          e:::::::eeeee::::::e
M::::::M   M:::::::M   M::::::M  aa::::::::::::a  l::::l  p:::::p     p:::::p h:::::h     h:::::h i::::i       t:::::t          e:::::::::::::::::e 
M::::::M    M:::::M    M::::::M a::::aaaa::::::a  l::::l  p:::::p     p:::::p h:::::h     h:::::h i::::i       t:::::t          e::::::eeeeeeeeeee  
M::::::M     MMMMM     M::::::Ma::::a    a:::::a  l::::l  p:::::p    p::::::p h:::::h     h:::::h i::::i       t:::::t    tttttte:::::::e           
M::::::M               M::::::Ma::::a    a:::::a l::::::l p:::::ppppp:::::::p h:::::h     h:::::hi::::::i      t::::::tttt:::::te::::::::e          
M::::::M               M::::::Ma:::::aaaa::::::a l::::::l p::::::::::::::::p  h:::::h     h:::::hi::::::i      tt::::::::::::::t e::::::::eeeeeeee  
M::::::M               M::::::M a::::::::::aa:::al::::::l p::::::::::::::pp   h:::::h     h:::::hi::::::i        tt:::::::::::tt  ee:::::::::::::e  
MMMMMMMM               MMMMMMMM  aaaaaaaaaa  aaaallllllll p::::::pppppppp     hhhhhhh     hhhhhhhiiiiiiii          ttttttttttt      eeeeeeeeeeeeee  
                                                          p:::::p                                                                                   
                                                          p:::::p                                                                                   
                                                         p:::::::p                                                                                  
                                                         p:::::::p                                                                                  
                                                         p:::::::p                                                                                  
                                                         ppppppppp                                                                                  
                                                                                                                                                                                                                                                                                                     
--]]

class "Malphite"

function Malphite:__init()

	if charName ~= "Malphite" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.baseMoveSpeed = myHero.ms

	Callback.Add("Tick", function() self:OnTick() end)
end

function Malphite:LoadSpells()

	Q = { range = 625,	delay = 0.25 }
	W = { range = 225 }
	E = { range = 300 }
	R = { range = 1000, width = 400 , delay = 0.25 , speed = 1835 }
end

function Malphite:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "rSettings", name = "R settings", type = MENU})
			Menu.Combo.rSettings:MenuElement({id = "useRks", name = "Use R to secure kills", value = true})
			Menu.Combo.rSettings:MenuElement({id = "useRcombo", name = "Use R Combo", value = true})
			Menu.Combo.rSettings:MenuElement({id = "useRcount", name = "Use R if enemy count > x", value = 3, min = 1, max = 6, step = 1})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Malphite:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range + R.width)

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		end
	end
end

function Malphite:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Utils:CastSpell(HK_Q, unit.pos, Q.range)
		end
	end
end

function Malphite:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		Control.CastSpell(HK_W)
	end
end

function Malphite:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Control.CastSpell(HK_E)
		end
	end
end

function Malphite:CastRcombo()

	if #Utils:GetEnemyHeroesInRange(myHero.pos, R.range + R.width) > 0 then
		for _, v in pairs(Utils:GetEnemyHeroesInRange(myHero.pos, R.range + R.width)) do
			local target = v
			if Utils:IsValid(target, myHero.pos, R.range + R.width) then
				if Utils:GetDistance(myHero.pos, target.pos) <= R.range then
					if #Utils:GetEnemyHeroesInRange(target.pos, R.width) >= Menu.Combo.rSettings.useRcount:Value() then
						Utils:CastSpell(HK_R, target.pos, R.range, R.delay)
						return
					end
				end
				if (self:GetComboDmg(target) > target.health and Menu.Combo.rSettings.useRcombo:Value()) or
					(self:GetRdmg(target) > target.health and Menu.Combo.rSettings.useRks:Value()) then
					Utils:CastSpell(HK_R, target.pos, R.range, R.delay)
					break
				end
			end
		end
	end
end

function Malphite:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Utils:IsReady(_R)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady then
		self:CastRcombo()
	end
end

function Malphite:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Malphite:GetComboDmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		local eLvl = myHero:GetSpellData(_E).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		if eLvl > 0 then
			totalDmg = totalDmg + self:GetEdmg(unit)
		end
		if Utils:IsReady(_R) then
			totalDmg = totalDmg + self:GetRdmg(unit)
		end
	end
	return totalDmg
end

function Malphite:GetQdmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 70, 120, 170, 220, 270 })[qLvl]
			local ap = myHero.ap * 0.6
			totalDmg = baseDmg + ap
		end
	end
	return Utils:CalculateMagicalDamage(unit, totalDmg)
end

function Malphite:GetEdmg(unit)

	local totalDmg = 0

	if unit then
		local eLvl = myHero:GetSpellData(_E).level
		if eLvl > 0 then
			local baseDmg = ({ 60, 95, 130, 165, 200 })[eLvl]
			local ap = myHero.ap * 0.6
			totalDmg = baseDmg + ap
		end
	end
	return Utils:CalculateMagicalDamage(unit, totalDmg)
end

function Malphite:GetRdmg(unit)

	local totalDmg = 0

	if unit then
		local rLvl = myHero:GetSpellData(_R).level
		if rLvl > 0 then
			local baseDmg = ({ 200, 300, 400 })[rLvl]
			local ap = myHero.ap
			totalDmg = baseDmg + ap
		end
	end
	return Utils:CalculateMagicalDamage(unit, totalDmg * 0.9)
end

function Malphite:OnDraw()
	-- TODO: ComboDmg next update #lazy
end

--[[
                                                                                                                    
                                                                                                                    
RRRRRRRRRRRRRRRRR                                                                                                   
R::::::::::::::::R                                                                                                  
R::::::RRRRRR:::::R                                                                                                 
RR:::::R     R:::::R                                                                                                
  R::::R     R:::::R    eeeeeeeeeeee    nnnn  nnnnnnnn       ggggggggg   ggggg  aaaaaaaaaaaaa   rrrrr   rrrrrrrrr   
  R::::R     R:::::R  ee::::::::::::ee  n:::nn::::::::nn    g:::::::::ggg::::g  a::::::::::::a  r::::rrr:::::::::r  
  R::::RRRRRR:::::R  e::::::eeeee:::::een::::::::::::::nn  g:::::::::::::::::g  aaaaaaaaa:::::a r:::::::::::::::::r 
  R:::::::::::::RR  e::::::e     e:::::enn:::::::::::::::ng::::::ggggg::::::gg           a::::a rr::::::rrrrr::::::r
  R::::RRRRRR:::::R e:::::::eeeee::::::e  n:::::nnnn:::::ng:::::g     g:::::g     aaaaaaa:::::a  r:::::r     r:::::r
  R::::R     R:::::Re:::::::::::::::::e   n::::n    n::::ng:::::g     g:::::g   aa::::::::::::a  r:::::r     rrrrrrr
  R::::R     R:::::Re::::::eeeeeeeeeee    n::::n    n::::ng:::::g     g:::::g  a::::aaaa::::::a  r:::::r            
  R::::R     R:::::Re:::::::e             n::::n    n::::ng::::::g    g:::::g a::::a    a:::::a  r:::::r            
RR:::::R     R:::::Re::::::::e            n::::n    n::::ng:::::::ggggg:::::g a::::a    a:::::a  r:::::r            
R::::::R     R:::::R e::::::::eeeeeeee    n::::n    n::::n g::::::::::::::::g a:::::aaaa::::::a  r:::::r            
R::::::R     R:::::R  ee:::::::::::::e    n::::n    n::::n  gg::::::::::::::g  a::::::::::aa:::a r:::::r            
RRRRRRRR     RRRRRRR    eeeeeeeeeeeeee    nnnnnn    nnnnnn    gggggggg::::::g   aaaaaaaaaa  aaaa rrrrrrr            
                                                                      g:::::g                                       
                                                          gggggg      g:::::g                                       
                                                          g:::::gg   gg:::::g                                       
                                                           g::::::ggg:::::::g                                       
                                                            gg:::::::::::::g                                        
                                                              ggg::::::ggg                                          
                                                                 gggggg                                             
--]]

class "Rengar"

function Rengar:__init()

	if charName ~= "Rengar" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.lastSpell = GetTickCount()
	self.isUlting = false

	Callback.Add("Tick", function() self:OnTick() end)
end

function Rengar:LoadSpells()

	Q = { range = 725 }
	W = { range = 450,	width = myHero:GetSpellData(_W).width,	speed = myHero:GetSpellData(_W).speed,	delay = 0.25 }
	E = { range = 1000,	width = myHero:GetSpellData(_E).width,	speed = myHero:GetSpellData(_E).speed,	delay = 0.25,
		collision = Collision:SetSpell(1000, myHero:GetSpellData(_E).speed, 0.25, 100, true)}
end

function Rengar:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "prio", name = "Priority",  drop = { "Q", "W", "E" }})
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useEcol", name = "Ignore E collision", value = true})
		Menu.Combo:MenuElement({id = "delay", name = "Spell Delays", value = 100, min = 0, max = 300, step = 10, tooltip = "increase if wrong skill gets empowered"})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Misc ---
		Menu.Misc:MenuElement({id = "useWcc", name = "Use W CC", value = true})
end

function Rengar:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)

	if Utils:HasBuff(myHero, "rengarr") then
		self.isUlting = true
	else
		self.isUlting = false
	end

	if Menu.Misc.useWcc:Value() then
		if Utils:IsImmobile(myHero) then
			if myHero.mana == 4 then
				Control.CastSpell(HK_W)
			end
		end
	end

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		end
	end
end

function Rengar:OnPostAttack()

	if Utils:IsReady(_Q) then
		if #Utils:GetEnemyHeroesInRange(myHero.pos, Q.range + myHero.boundingRadius * 2) > 0 then
			local prio = Menu.Combo.prio:Value()
			if (Menu.Combo.useQ:Value() and Orb.combo) or (Menu.Harass.useQ:Value() and Orb.harass) then
				if myHero.mana < 4 or prio == 1 then
					Control.CastSpell(HK_Q)
					self.lastSpell = GetTickCount()
				end
			end
		end
	end
end

function Rengar:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		local p = myHero.pathing
		if p.hasMovePath and p.isDashing then
			Control.CastSpell(HK_Q)
			self.lastSpell = GetTickCount()
		end
		if #Utils:GetEnemyHeroesInRange(myHero.pos, Q.range + myHero.boundingRadius * 2) > 0 then
			Control.CastSpell(HK_Q)
			self.lastSpell = GetTickCount()
		end
	end
end

function Rengar:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) then
		local prio = Menu.Combo.prio:Value()
		if myHero.mana < 4 or prio == 2 then
			Control.CastSpell(HK_W)
			self.lastSpell = GetTickCount()
		end
	end
end

function Rengar:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		local prio = Menu.Combo.prio:Value()
		if myHero.mana < 4 or prio == 3 or unit.health < self:GetComboDmg(unit) then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, E.speed, E.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
						local col, units = E.collision:__GetCollision(myHero.pos, nPred, 5)
						if Menu.Combo.useEcol:Value() then
							Utils:CastSpell(HK_E, nPred, E.range)
						else							
							if not col then
								Utils:CastSpell(HK_E, nPred, E.range)
							end
						end
					end					
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
						local col, units = E.collision:__GetCollision(myHero.pos, aimPos, 5)
						if Menu.Combo.useEcol:Value() then
							Utils:CastSpell(HK_E, aimPos, E.range)
						else							
							if not col then
								Utils:CastSpell(HK_E, aimPos, E.range)
							end
						end
					end
				end
				if hitChance and aimPos2 then
					if (hitChance >= minAccuracy) or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
						if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < E.range then
							local col, units = E.collision:__GetCollision(myHero.pos, aimPos2, 5)
							if Menu.Combo.useEcol:Value() then
								Utils:CastSpell(HK_E, aimPos2, E.range)
							else							
								if not col then
									Utils:CastSpell(HK_E, aimPos2, E.range)
								end
							end
						end
					end
				end
			end
		end
	end
end

function Rengar:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local delay = Menu.Combo.delay:Value()
	local wait = (myHero.mana == 3 or myHero.mana == 4) and self.lastSpell + delay > GetTickCount()

	if qReady and self.qTarget and not wait then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget and not wait then
		self:CastWcombo(self.wTarget)
	end

	if not self.isUlting then
		if eReady and self.eTarget and not wait then
			self:CastEcombo(self.eTarget)
		end
	end
end

function Rengar:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)
	local wait = (myHero.mana == 3 or myHero.mana == 4) and self.lastSpell + delay > GetTickCount()

	if qReady and self.qTarget and not wait then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget and not wait then
		self:CastWcombo(self.wTarget)
	end

	if not self.isUlting then
		if eReady and self.eTarget and not wait then
			self:CastEcombo(self.eTarget)
		end
	end
end

function Rengar:GetComboDmg(unit)

	local totalDmg = 0
	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		local wLvl = myHero:GetSpellData(_W).level
		local eLvl = myHero:GetSpellData(_E).level
		local rLvl = myHero:GetSpellData(_R).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		if wLvl > 0 then
			totalDmg = totalDmg + self:GetWdmg(unit)
		end
		if eLvl > 0 then
			totalDmg = totalDmg + self:GetEdmg(unit)
		end
		if rLvl > 0 and Utils:IsReady(_R) then
			totalDmg = totalDmg + myHero.totalDamage
		end
	end
	return totalDmg
end

function Rengar:GetQdmg(unit)

	local totalDmg = 0
	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		if qLvl > 0 then
			local baseDmg = ({ 30, 60, 90, 120, 150 })[qLvl]
			local ap = myHero.ap * 0.6
			totalDmg = baseDmg + ap
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

function Rengar:GetWdmg(unit)

	local totalDmg = 0
	if unit then
		local wLvl = myHero:GetSpellData(_W).level
		if wLvl > 0 then
			local baseDmg = ({ 50, 80, 110, 140, 170  })[wLvl]
			local apMod = .8
			local totalAp = myHero.ap * apMod
			totalDmg = baseDmg + totalAp
		end
	end
	return Utils:CalculateMagicalDamage(unit, totalDmg)
end

function Rengar:GetEdmg(unit)

	local totalDmg = 0
	if unit then
		local eLvl = myHero:GetSpellData(_E).level
		if eLvl > 0 then
			local baseDmg = ({ 55, 100, 145, 190, 235 })[eLvl]
			local adMod = .8
			local totalAd = myHero.bonusDamage * adMod
			totalDmg = baseDmg + totalAd
		end
	end
	return Utils:CalculatePhysicalDamage(unit, totalDmg)
end

--[[
                                                                                                                                
LLLLLLLLLLL                                 BBBBBBBBBBBBBBBBB   lllllll                                                         
L:::::::::L                                 B::::::::::::::::B  l:::::l                                                         
L:::::::::L                                 B::::::BBBBBB:::::B l:::::l                                                         
LL:::::::LL                                 BB:::::B     B:::::Bl:::::l                                                         
  L:::::L                   eeeeeeeeeeee      B::::B     B:::::B l::::l   aaaaaaaaaaaaa   nnnn  nnnnnnnn        cccccccccccccccc
  L:::::L                 ee::::::::::::ee    B::::B     B:::::B l::::l   a::::::::::::a  n:::nn::::::::nn    cc:::::::::::::::c
  L:::::L                e::::::eeeee:::::ee  B::::BBBBBB:::::B  l::::l   aaaaaaaaa:::::a n::::::::::::::nn  c:::::::::::::::::c
  L:::::L               e::::::e     e:::::e  B:::::::::::::BB   l::::l            a::::a nn:::::::::::::::nc:::::::cccccc:::::c
  L:::::L               e:::::::eeeee::::::e  B::::BBBBBB:::::B  l::::l     aaaaaaa:::::a   n:::::nnnn:::::nc::::::c     ccccccc
  L:::::L               e:::::::::::::::::e   B::::B     B:::::B l::::l   aa::::::::::::a   n::::n    n::::nc:::::c             
  L:::::L               e::::::eeeeeeeeeee    B::::B     B:::::B l::::l  a::::aaaa::::::a   n::::n    n::::nc:::::c             
  L:::::L         LLLLLLe:::::::e             B::::B     B:::::B l::::l a::::a    a:::::a   n::::n    n::::nc::::::c     ccccccc
LL:::::::LLLLLLLLL:::::Le::::::::e          BB:::::BBBBBB::::::Bl::::::la::::a    a:::::a   n::::n    n::::nc:::::::cccccc:::::c
L::::::::::::::::::::::L e::::::::eeeeeeee  B:::::::::::::::::B l::::::la:::::aaaa::::::a   n::::n    n::::n c:::::::::::::::::c
L::::::::::::::::::::::L  ee:::::::::::::e  B::::::::::::::::B  l::::::l a::::::::::aa:::a  n::::n    n::::n  cc:::::::::::::::c
LLLLLLLLLLLLLLLLLLLLLLLL    eeeeeeeeeeeeee  BBBBBBBBBBBBBBBBB   llllllll  aaaaaaaaaa  aaaa  nnnnnn    nnnnnn    cccccccccccccccc
                                                                                                                                                                                                                                                               
--]]

class "Leblanc"

function Leblanc:__init()

	if charName ~= "Leblanc" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
end

function Leblanc:LoadSpells()

	Q = { range = 700,				delay = .25 }
	W = { range = 600, width = 250,	delay = .25,	speed = 1500 }
	E = { range = 925, width = 60,	delay = .25,	speed = 1800,	collision = Collision:SetSpell(925, 1500, 0.25, 55, true) }
	R = { range = 925, width = 60,	delay = .25,	speed = 1800,	collision = Collision:SetSpell(925, 1500, 0.25, 55, true), name = myHero:GetSpellData(_R).name:lower() }
end

function Leblanc:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})
		--Menu.Combo:MenuElement({id = "useRq", name = "Use R-Q", value = true})
		--Menu.Combo:MenuElement({id = "useRw", name = "Use R-W", value = true})
		--Menu.Combo:MenuElement({id = "useRe", name = "Use R-E", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Harass:MenuElement({id = "useR", name = "Use R", value = true})
end

function Leblanc:OnTick()

	self:UpdateRstate()
	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Leblanc:UpdateRstate()
	
	R.name = myHero:GetSpellData(_R).name:lower()

	if R.name == "leblancr" then
		R.range = 0
		R.width = 0
		R.speed = 0
	elseif R.name == "leblancrq" then
		R.range = Q.range
	elseif R.name == "leblancrw" then
		R.range = W.range
		R.width = W.width
		R.speed = W.speed
	elseif R.name == "leblancre" then
		R.range = E.range
		R.width = E.width
		R.speed = E.speed
		R.collision = E.collision
	end
end

function Leblanc:HasMark(unit)

	return Utils:HasBuff(unit, "leblancqmark")
end

function Leblanc:HasChain(unit)
	
	return Utils:HasBuff(unit, "leblance")
end

function Leblanc:CastQcombo(unit, key)

	if Utils:IsValid(unit, myHero.pos, Q.range) then
		Control.CastSpell(key, unit)
	end
end

function Leblanc:CastWcombo(unit, key)

	local wName = myHero:GetSpellData(_W).name:lower()
	if Utils:IsValid(unit, myHero.pos, W.range) and myHero.attackData.state ~= STATE_WINDUP and wName == "leblancw" then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < W.range then
					Utils:CastSpell(key, nPred, W.range, W.delay)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < W.range then
					Utils:CastSpell(key, aimPos, W.range, W.delay)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < W.range then
						Utils:CastSpell(key, aimPos2, W.range, W.delay)
					end
				end
			end
		end
	end
end

function Leblanc:CastEcombo(unit, key)

	if Utils:IsValid(unit, myHero.pos, E.range) and myHero.attackData.state ~= STATE_WINDUP then
		if Utils:GetDistance(myHero.pos, unit.pos) < myHero.boundingRadius * 2 then return end
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, E.speed, E.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
					local col, units = E.collision:__GetCollision(myHero.pos, nPred, 5)						
					if not col then
						Utils:CastSpell(key, nPred, E.range, E.delay)
					end
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
					local col, units = E.collision:__GetCollision(myHero.pos, aimPos, 5)					
					if not col then
						Utils:CastSpell(key, aimPos, E.range, E.delay)
					end
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < E.range then
						local col, units = E.collision:__GetCollision(myHero.pos, aimPos2, 5)							
						if not col then
							Utils:CastSpell(key, aimPos2, E.range, E.delay)
						end
					end
				end
			end
		end
	end
end

function Leblanc:CastRcombo(unit)
	-- maybe playing around with different rotations one day. for now its adaptive
	if R.name == "leblancrq" then
		--if Menu.Combo.useRq:Value() then
			self:CastQcombo(unit, HK_R)
		--end
	elseif R.name == "leblancrw" and self:HasMark(unit) then
		--if Menu.Combo.useRw:Value() then
			self:CastWcombo(unit, HK_R)
		--end
	elseif R.name == "leblancre" and self:HasChain(unit) or self:HasMark(unit) then
		--if Menu.Combo.useRe:Value() then
			self:CastEcombo(unit, HK_R)
		--end
	end
end

function Leblanc:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget, HK_Q)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget, HK_W)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget, HK_E)
	end
end

function Leblanc:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Harass.useR:Value() and Utils:IsReady(_R)

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget, HK_Q)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget, HK_W)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget, HK_E)
	end
end

--[[                                                                              
                                                                                    
PPPPPPPPPPPPPPPPP                            kkkkkkkk                               
P::::::::::::::::P                           k::::::k                               
P::::::PPPPPP:::::P                          k::::::k                               
PP:::::P     P:::::P                         k::::::k                               
  P::::P     P:::::Pyyyyyyy           yyyyyyy k:::::k    kkkkkkk    eeeeeeeeeeee    
  P::::P     P:::::P y:::::y         y:::::y  k:::::k   k:::::k   ee::::::::::::ee  
  P::::PPPPPP:::::P   y:::::y       y:::::y   k:::::k  k:::::k   e::::::eeeee:::::ee
  P:::::::::::::PP     y:::::y     y:::::y    k:::::k k:::::k   e::::::e     e:::::e
  P::::PPPPPPPPP        y:::::y   y:::::y     k::::::k:::::k    e:::::::eeeee::::::e
  P::::P                 y:::::y y:::::y      k:::::::::::k     e:::::::::::::::::e 
  P::::P                  y:::::y:::::y       k:::::::::::k     e::::::eeeeeeeeeee  
  P::::P                   y:::::::::y        k::::::k:::::k    e:::::::e           
PP::::::PP                  y:::::::y        k::::::k k:::::k   e::::::::e          
P::::::::P                   y:::::y         k::::::k  k:::::k   e::::::::eeeeeeee  
P::::::::P                  y:::::y          k::::::k   k:::::k   ee:::::::::::::e  
PPPPPPPPPP                 y:::::y           kkkkkkkk    kkkkkkk    eeeeeeeeeeeeee  
                          y:::::y                                                   
                         y:::::y                                                    
                        y:::::y                                                     
                       y:::::y                                                      
                      yyyyyyy                                                                                                                                           
                                                                                    
--]]

class "Pyke"

function Pyke:__init()

	if charName ~= "Pyke" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.lastQ = 0
	self.chargingQ = false
	self.qTime = 0
	self.lastR = 0

	Callback.Add("Tick", function() self:OnTick() end)
end

function Pyke:LoadSpells()

	Q = { range = 400, minRange = 400,	maxRange = 1025,	delay = 0.25,	speed = 2000,	width = 70,	collision = Collision:SetSpell(400, 2000, 0.25, 70, true) }
	W = { range = 900 }
	E = { range = 525,										delay = 0.25,	speed = 1700,	width = 150 }
	R = { range = 900,										delay = 0.25,	speed = _huge,	width = 250 }
end

function Pyke:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})
		Menu.Combo:MenuElement({id = "rDelay", name = "R reactivation delay", value = 500, min = 0, max = 1000, step = 50, tooltip = "(ms)" })
	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = false})
end

function Pyke:OnTick()

	self.qPullTarget = Utils:GetTarget(Q.maxRange)
	self.qTarget = Utils:GetTarget(Q.minRange)
	self.eTarget = Utils:GetTarget(E.range - myHero.boundingRadius * 2)
	self.rTarget = Utils:GetTarget(R.range)
	self:UpdateQstate()

	if self.chargingQ then
		Orb:DisableAttacks(true)
	else
		Orb:DisableAttacks(false)
	end

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		end
	end
end

function Pyke:UpdateQstate()

	self.qTime = LocalGameTimer() - self.lastQ

	if self.chargingQ and self.qTime > 0.5 then
		Q.range = 400 + (140 * (LocalGameTimer() - (self.lastQ + 0.5))) * 10
		if Q.range > Q.maxRange then
			Q.range = Q.maxRange
			
		end
	end

	Q.collision = Collision:SetSpell(Q.range, 2000, 0.25, 70, true)

	local hasQbuff = Utils:HasBuff(myHero,"pykeq") or Utils:IsParticleInRange("pyke_base_q_channel", myHero.pos, myHero.boundingRadius)

	if hasQbuff and not self.chargingQ then
		self.chargingQ = true
	end

	if self.chargingQ and not hasQbuff then
		self.chargingQ = false
		Q.range = 400
		if Control.IsKeyDown(HK_Q) == true then
			Q.range = 400
			Control.KeyUp(HK_Q)
		end
	end
	if Control.IsKeyDown(HK_Q) == true and not self.chargingQ then
		DelayAction(function()
			if Control.IsKeyDown(HK_Q) == true and not self.chargingQ then
				Control.KeyUp(HK_Q)
			end
		end,0.25)
	end
	if Control.IsKeyDown(HK_Q) == true and not Utils:IsReady(_Q) then
		DelayAction(function()
			if Control.IsKeyDown(HK_Q) == true then
				Q.range = 400
				Control.KeyUp(HK_Q)
			end
		end,0.01)
	end
end

function Pyke:CastQ(unit)

	if Utils:IsValid(unit, myHero.pos, Q.minRange) then
		if not self.chargingQ and self.qTime < 0.5 then
			Utils:CastSpell(HK_Q, unit.pos, Q.minRange)
		end
	end
end

function Pyke:CastQstart(unit)

	if Utils:IsValid(unit, myHero.pos, Q.maxRange) then
		if not self.chargingQ then
			local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
			if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.maxRange then
				local col, units = Q.collision:__GetCollision(myHero.pos, nPred, 5)
				if not col then
					Control.KeyDown(HK_Q)
					self.lastQ = LocalGameTimer()
				end
			end
		end
	end
end

function Pyke:CastQcharged(unit)

	local chargeReady = self.qTime > 0.75 and self.qTime < 3

	if Utils:IsValid(unit, myHero.pos, Q.range) and self.chargingQ and chargeReady then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.range then
					local col, units = Q.collision:__GetCollision(myHero.pos, nPred, 5)
					if not col then
						Utils:CastSpellCharged(HK_Q, nPred, Q.range)
					end
				end
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < Q.range then
					local col, units = Q.collision:__GetCollision(myHero.pos, aimPos, 5)							
					if not col then
						Utils:CastSpellCharged(HK_Q, aimPos, Q.range)
					end
				end
			end
			if hitChance and aimPos2 then
				if (hitChance >= minAccuracy) then --or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < Q.range then
						local col, units = Q.collision:__GetCollision(myHero.pos, aimPos2, 5)					
						if not col then
							Utils:CastSpellCharged(HK_Q, aimPos2, Q.range)
						end
					end
				end
			end
		end
	end
end

function Pyke:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, E.speed, E.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < E.range then
					Utils:CastSpell(HK_E, nPred, E.range)
				end
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < E.range then
					Utils:CastSpell(HK_E, aimPos, E.range)
				end
			end
			if hitChance and aimPos2 then
				if (hitChance >= minAccuracy) then --or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < E.range then
						Utils:CastSpell(HK_E, aimPos2, E.range)
					end
				end
			end
		end
	end
end

function Pyke:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) then
		if self:GetRdmg(unit) > unit.health and LocalGameTimer() - self.lastR > Menu.Combo.rDelay:Value() * 0.001 then
			local nPred = Utils:GetPred(unit, R.speed, R.delay)
			if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < R.range then
				Utils:CastSpell(HK_R, nPred, R.range)
				self.lastR = LocalGameTimer()
			end
		end
	end
end

function Pyke:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady then
		if self.qPullTarget then
			if not self.chargingQ then
				self:CastQstart(self.qPullTarget)
			else
				self:CastQcharged(self.qPullTarget)
			end
		end

		if self.qTarget and not self.chargingQ then
			self:CastQ(self.qTarget)
		end
	end

	if eReady and self.eTarget and not self.chargingQ then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget and not self.chargingQ then
		self:CastRcombo(self.rTarget)
	end
end

function Pyke:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady then
		if self.qPullTarget then
			if not self.chargingQ then
				self:CastQstart(self.qPullTarget)
			else
				self:CastQcharged(self.qPullTarget)
			end
		end

		if self.qTarget and not self.chargingQ then
			self:CastQ(self.qTarget)
		end
	end

	if eReady and self.eTarget and not self.chargingQ then
		self:CastEcombo(self.eTarget)
	end
end

function Pyke:GetRdmg(unit)

	local total = 0
	if myHero.levelData.lvl >= 6 then
		local rLvl = myHero.levelData.lvl - 5
		local raw = ({ 190, 240, 290, 340, 390, 440,475, 510, 545, 580, 615, 635, 655 })[rLvl]
		local ad = myHero.bonusDamage * 0.6
		total = raw + ad
	end
	return total - 5
end

--[[
                                                                                                                                                                                                                               
KKKKKKKKK    KKKKKKK                  lllllll   iiii                            tttt                            
K:::::::K    K:::::K                  l:::::l  i::::i                        ttt:::t                            
K:::::::K    K:::::K                  l:::::l   iiii                         t:::::t                            
K:::::::K   K::::::K                  l:::::l                                t:::::t                            
KK::::::K  K:::::KKK  aaaaaaaaaaaaa    l::::l iiiiiii     ssssssssss   ttttttt:::::ttttttt      aaaaaaaaaaaaa   
  K:::::K K:::::K     a::::::::::::a   l::::l i:::::i   ss::::::::::s  t:::::::::::::::::t      a::::::::::::a  
  K::::::K:::::K      aaaaaaaaa:::::a  l::::l  i::::i ss:::::::::::::s t:::::::::::::::::t      aaaaaaaaa:::::a 
  K:::::::::::K                a::::a  l::::l  i::::i s::::::ssss:::::stttttt:::::::tttttt               a::::a 
  K:::::::::::K         aaaaaaa:::::a  l::::l  i::::i  s:::::s  ssssss       t:::::t              aaaaaaa:::::a 
  K::::::K:::::K      aa::::::::::::a  l::::l  i::::i    s::::::s            t:::::t            aa::::::::::::a 
  K:::::K K:::::K    a::::aaaa::::::a  l::::l  i::::i       s::::::s         t:::::t           a::::aaaa::::::a 
KK::::::K  K:::::KKKa::::a    a:::::a  l::::l  i::::i ssssss   s:::::s       t:::::t    tttttta::::a    a:::::a 
K:::::::K   K::::::Ka::::a    a:::::a l::::::li::::::is:::::ssss::::::s      t::::::tttt:::::ta::::a    a:::::a 
K:::::::K    K:::::Ka:::::aaaa::::::a l::::::li::::::is::::::::::::::s       tt::::::::::::::ta:::::aaaa::::::a 
K:::::::K    K:::::K a::::::::::aa:::al::::::li::::::i s:::::::::::ss          tt:::::::::::tt a::::::::::aa:::a
KKKKKKKKK    KKKKKKK  aaaaaaaaaa  aaaalllllllliiiiiiii  sssssssssss              ttttttttttt    aaaaaaaaaa  aaaa
                                                                                                          
--]]

class "Kalista"

function Kalista:__init()

	if charName ~= "Kalista" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.lastAttack = GetTickCount()
	self.soulBound = nil

	Callback.Add("Tick", function() self:OnTick() end)
end

function Kalista:LoadSpells()

	Q = { range = 1150,	delay = 0.4, speed = 1200,	width = 40, collision = Collision:SetSpell(1150, 1200, 0.25, 40, true) }
	W = { range = 5000 }
	E = { range = 1000 }
	R = { range = 1000, speed = 500 }
end

function Kalista:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Clear", name = "Clear Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "eSettings", name = "E settings", type = MENU})
			Menu.Combo.eSettings:MenuElement({id = "kill", name = "Use E to ks", value = true})
			Menu.Combo.eSettings:MenuElement({id = "flee", name = "Use E on fleeing target", value = true})
			Menu.Combo.eSettings:MenuElement({id = "fleeHp", name = "Fleeing Target Hp", value = 20, min = 0, max = 100, step = 1})
			Menu.Combo.eSettings:MenuElement({id = "harass", name = "Harass with minions", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R to save allies", value = true})
		Menu.Combo:MenuElement({id = "useRhp", name = "Ally Hp percent", value = 10, min = 0, max = 100, step = 1})
		Menu.Combo:MenuElement({id = "jump", name = "Attack minions to gapclose", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "eSettings", name = "E settings", type = MENU})
			Menu.Harass.eSettings:MenuElement({id = "kill", name = "Use E to ks", value = true})
			Menu.Harass.eSettings:MenuElement({id = "flee", name = "Use E on fleeing target", value = true})
			Menu.Harass.eSettings:MenuElement({id = "fleeHp", name = "Fleeing Target Hp", value = 20, min = 0, max = 100, step = 1})
			Menu.Harass.eSettings:MenuElement({id = "harass", name = "Harass with minions", value = true})
		Menu.Harass:MenuElement({id = "jump", name = "Attack minions to gapclose", value = true})

	--- Clear ---
		Menu.Clear:MenuElement({id = "useE", name = "Use E", value = true})
end

function Kalista:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.eTarget = Utils:GetTarget(E.range)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R) 

	if self.soulBound then
		if rReady then
			self:CastRsave(self.soulBound, Menu.Combo.useRhp:Value())
		end
	else
		for i = 1, #Utils:GetAllyHeroesInRange(myHero.pos, R.range) do
			local ally = Utils:GetAllyHeroesInRange(myHero.pos, R.range)[i]
			if ally and Utils:HasBuff(ally, "kalistacoopstrikeally") then
				self.soulBound = ally
				break
			end
		end
	end

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		elseif Orb.laneClear or Orb.jungleClear then
			self:Clear()
		end
	end
end

function Kalista:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) and myHero.attackData.state ~= STATE_WINDUP then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.range then
					local col, units = Q.collision:__GetCollision(myHero.pos, nPred, 5)						
					if not col then
						Utils:CastSpell(HK_Q, nPred, Q.range, Q.delay)
					end
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < Q.range then
					local col, units = Q.collision:__GetCollision(myHero.pos, aimPos, 5)					
					if not col then
						Utils:CastSpell(HK_Q, aimPos, Q.range, Q.delay)
					end
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < Q.range then
						local col, units = Q.collision:__GetCollision(myHero.pos, aimPos2, 5)							
						if not col then
							Utils:CastSpell(HK_Q, aimPos2, Q.range, Q.delay)
						end
					end
				end
			end
		end
	end
end

function Kalista:CastQeCombo(unit)

	if #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) > 0 then
		for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) do
			local m = Utils:GetEnemyMinionsInRange(myHero.pos, Q.range)[i]
			if m and Utils:IsValid(m, myHero.pos, Q.range) and self:GetStacks(m) > 0 and self:GetQdmg(m) > m.health then
				local myPos = myHero.pos
				local unitPos = unit.pos
				local mPos = m.pos
				for i = unit.boundingRadius, Q.range do
					local posExtended = myPos:Extended(mPos, i)
					if Utils:GetDistance(unit.pos, posExtended) <= Q.width then
						local col, units = Q.collision:__GetCollision(mPos, unitPos, 5)
						local col2, units2 = Q.collision:__GetCollision(myPos, mPos, 5)
						if not col and not col2 then
							Utils:CastSpell(HK_Q, m.pos, Q.range)
						end
					end
				end		
			end
		end
	end
end

function Kalista:AttackMinion()

	if self.qTarget and myHero.attackData.state ~= STATE_WINDUP then
		if Utils:IsValid(self.qTarget, myHero.pos, Q.range) and Utils:GetDistance(self.qTarget.pos, myHero.pos) > myHero.range then
			if #Utils:GetEnemyMinionsInRange(myHero.pos, myHero.range) > 0 then
				for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, myHero.range) do
					local m = Utils:GetEnemyMinionsInRange(myHero.pos, myHero.range)[i]
					if m and Utils:IsValid(m, myHero.pos, myHero.range) and GetTickCount() > self.lastAttack + 100 then
						Control.Attack(m)
						self.lastAttack = GetTickCount()
					end
				end
			end
		end
	end
end

function Kalista:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local eReady = Utils:IsReady(_E)
	local jump = Menu.Combo.jump:Value()

	if jump then
		if self.qTarget and myHero.attackData.state ~= STATE_WINDUP then
			self:AttackMinion()
			if qReady then
				self:CastQcombo(self.qTarget)
			end
		end
	end

	if eReady then
		if self.eTarget then
			if Menu.Combo.eSettings.kill:Value() then
				if self:GetEdmg(self.eTarget) > self.eTarget.health then
					Control.CastSpell(HK_E)
				end
			end
			if Menu.Combo.eSettings.flee:Value() then
				self:CastEflee(self.eTarget, Menu.Combo.eSettings.fleeHp:Value())
			end
		end
		if Menu.Combo.eSettings.harass:Value() then
			self:MinionHarass()
		end
	end

	if qReady and eReady and self.eTarget then
		self:CastQeCombo(self.eTarget)
	end
end

function Kalista:CastEflee(unit, hp)

	if Utils:IsValid(unit, myHero.pos, E.range) and Utils:GetHpPercent(unit) <= hp and self:GetStacks(unit) > 0 then
		local pred = unit:GetPrediction(unit.ms, .25)
		if pred:To2D().onScreen and Utils:GetDistance(pred, myHero.pos) > E.range then
			Control.CastSpell(HK_E)
		end
	end
end

function Kalista:CastRsave(ally, hp)
	
	if not ally then return end
	if Utils:IsValid(ally, myHero.pos, E.range) and Utils:GetHpPercent(ally) <= hp then
		Control.CastSpell(HK_R)
	end
end

function Kalista:MinionHarass()

	if #Utils:GetEnemyMinionsInRange(myHero.pos, E.range) > 0 and #Utils:GetEnemyHeroesInRange(myHero.pos, E.range) > 0 then
		for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, E.range) do
			local m = Utils:GetEnemyMinionsInRange(myHero.pos, E.range)[i]
			if m and self:GetStacks(m) > 0 and self:GetEdmg(m) * 0.5 > m.health then
				Control.CastSpell(HK_E)
			end
		end
	end
end

function Kalista:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local eReady = Utils:IsReady(_E)
	local jump = Menu.Harass.jump:Value()

	if jump then
		if self.qTarget and myHero.attackData.state ~= STATE_WINDUP then
			self:AttackMinion()
			if qReady then
				self:CastQcombo(self.qTarget)
			end
		end
	end

	if eReady then
		if self.eTarget then
			if Menu.Harass.eSettings.kill:Value() then
				if self:GetEdmg(self.eTarget) > self.eTarget.health then
					Control.CastSpell(HK_E)
				end
			end
			if Menu.Harass.eSettings.flee:Value() then
				self:CastEflee(self.eTarget, Menu.Harass.eSettings.fleeHp:Value())
			end
		end
		if Menu.Harass.eSettings.harass:Value() then
			self:MinionHarass()
		end
	end

	if qReady and eReady and self.eTarget then
		self:CastQeCombo(self.eTarget)
	end
end

function Kalista:Clear()

	local eReady = Menu.Clear.useE:Value() and Utils:IsReady(_E)

	if eReady then
		if #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) > 0 then
			for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) do
				local m = Utils:GetEnemyMinionsInRange(myHero.pos, Q.range)[i]
				if m and Utils:IsValid(m, myHero.pos, Q.range) and self:GetStacks(m) > 0 and self:GetQdmg(m) * 0.5 > m.health then
					Control.CastSpell(HK_E)
				end
			end
		end
	end
end

function Kalista:GetStacks(unit)

	local stacks = 0
	for i = 1, unit.buffCount do
		local buff = unit:GetBuff(i)
		if buff and buff.count > 0 and buff.name:lower() == "kalistaexpungemarker" then
			stacks = buff.count
		end
	end
	return stacks
end

function Kalista:GetQdmg(unit)

	if not unit then return 0 end
	local total = 0
	local qLvl = myHero:GetSpellData(_Q).level
	if qLvl > 0 then
		local raw = ({ 10, 70, 130, 190, 250 })[qLvl]
		local bonusDmg = myHero.totalDamage
		total = (raw + bonusDmg)
	end
	return Utils:CalculatePhysicalDamage(unit, total)
end

function Kalista:GetEdmg(unit)

	if not unit then return 0 end
	local total = 0
	local eLvl = myHero:GetSpellData(_E).level
	if eLvl > 0 then
		local raw = ({ 10, 14, 19, 25, 32 })[eLvl]
		local m = ({ .2, .225, .25, .275, .3 })[eLvl]
		local bonusDmg = m * myHero.totalDamage
		total = (raw + bonusDmg) * self:GetStacks(unit)
	end
	return Utils:CalculatePhysicalDamage(unit, total)
end

--[[
                                                                                                                                 
               AAA                                          tttt                                                                   
              A:::A                                      ttt:::t                                                                   
             A:::::A                                     t:::::t                                                                   
            A:::::::A                                    t:::::t                                                                   
           A:::::::::A             aaaaaaaaaaaaa   ttttttt:::::ttttttt    rrrrr   rrrrrrrrr      ooooooooooo   xxxxxxx      xxxxxxx
          A:::::A:::::A            a::::::::::::a  t:::::::::::::::::t    r::::rrr:::::::::r   oo:::::::::::oo  x:::::x    x:::::x 
         A:::::A A:::::A           aaaaaaaaa:::::a t:::::::::::::::::t    r:::::::::::::::::r o:::::::::::::::o  x:::::x  x:::::x  
        A:::::A   A:::::A                   a::::a tttttt:::::::tttttt    rr::::::rrrrr::::::ro:::::ooooo:::::o   x:::::xx:::::x   
       A:::::A     A:::::A           aaaaaaa:::::a       t:::::t           r:::::r     r:::::ro::::o     o::::o    x::::::::::x    
      A:::::AAAAAAAAA:::::A        aa::::::::::::a       t:::::t           r:::::r     rrrrrrro::::o     o::::o     x::::::::x     
     A:::::::::::::::::::::A      a::::aaaa::::::a       t:::::t           r:::::r            o::::o     o::::o     x::::::::x     
    A:::::AAAAAAAAAAAAA:::::A    a::::a    a:::::a       t:::::t    tttttt r:::::r            o::::o     o::::o    x::::::::::x    
   A:::::A             A:::::A   a::::a    a:::::a       t::::::tttt:::::t r:::::r            o:::::ooooo:::::o   x:::::xx:::::x   
  A:::::A               A:::::A  a:::::aaaa::::::a       tt::::::::::::::t r:::::r            o:::::::::::::::o  x:::::x  x:::::x  
 A:::::A                 A:::::A  a::::::::::aa:::a        tt:::::::::::tt r:::::r             oo:::::::::::oo  x:::::x    x:::::x 
AAAAAAA                   AAAAAAA  aaaaaaaaaa  aaaa          ttttttttttt   rrrrrrr               ooooooooooo   xxxxxxx      xxxxxxx
                                                                                                                              
--]]

class "Aatrox"

function Aatrox:__init()

	if charName ~= "Aatrox" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.qState = 1
	self.lastQ = GetTickCount()

	Callback.Add("Tick", function() self:OnTick() end)
end

function Aatrox:LoadSpells()

	Q = { range = 650,	delay = 0.25 }
	W = { range = 825,	width = 80,	speed = 500, delay = 0.25, collision = Collision:SetSpell(825, 500, 0.25, 80, true) }
	E = { range = 300 }
	R = { }
end

function Aatrox:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useRhp", name = "HP to use R", value = 40, min = 0, max = 100, step = 1, tooltip = "set to 0 to disable"})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
end

function Aatrox:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(Q.range + E.range)

	local qName = myHero:GetSpellData(_Q).name:lower()

	if Utils:GetHpPercent(myHero) <= Menu.Combo.useRhp:Value() and #Utils:GetEnemyHeroesInRange(myHero.pos, 800) > 0 then
		if Utils:IsReady(_R) then
			Control.CastSpell(HK_R)
		end
	end

	if myHero.activeSpell.valid and myHero.activeSpellSlot == 0 then
		self.lastQ = GetTickCount()
	end

	if qName == "aatroxq" then
		self.qState = 1
	elseif qName == "aatroxq2" then
		self.qState = 2
	else
		self.qState = 3
	end

	if self.qState == 1 then
		Q.range = 650
	elseif self.qState == 2 then
		Q.range = 525
	else
		Q.range = 500
	end

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Aatrox:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) and not 
		(myHero.activeSpell.valid or myHero.pathing.isDashing) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Utils:CastSpell(HK_Q, unit.pos, Q.range, Q.delay)
		end
	end
end

function Aatrox:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) and myHero.attackData.state ~= STATE_WINDUP then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < W.range then
					local col, units = W.collision:__GetCollision(myHero.pos, nPred, 5)						
					if not col then
						Utils:CastSpell(HK_W, nPred, W.range, W.delay)
					end
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < W.range then
					local col, units = W.collision:__GetCollision(myHero.pos, aimPos, 5)					
					if not col then
						Utils:CastSpell(HK_W, aimPos, W.range, W.delay)
					end
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < W.range then
						local col, units = W.collision:__GetCollision(myHero.pos, aimPos2, 5)							
						if not col then
							Utils:CastSpell(HK_W, aimPos2, W.range, W.delay)
						end
					end
				end
			end
		end
	end
end

function Aatrox:CastEcombo(unit)

	if myHero.attackData.state ~= STATE_WINDUP and self.lastQ + 500 < GetTickCount() and not myHero.activeSpell.isChanneling then
		if Utils:IsValid(unit, myHero.pos, E.range + Q.range) then
			local extPos = unit.pos:Extended(myHero.pos, Utils:GetDistance(myHero.pos, unit.pos) + 300)
			if (self.qState == 1 and Utils:IsReady(_Q)) or self.qState == 2 then
				if Utils:GetDistance(myHero.pos, unit.pos) > Q.range then
					Utils:CastSpell(HK_E, unit.pos, E.range)
				else
					if Utils:GetDistance(myHero.pos, unit.pos) < Q.range * .75 then
						Utils:CastSpell(HK_E, extPos, E.range)
					end
				end
			else
				if Utils:GetDistance(myHero.pos, unit.pos) > Q.range * .5 then
					Utils:CastSpell(HK_E, unit.pos, E.range)							
				end
			end
		end
	end
end

function Aatrox:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end 

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Aatrox:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

--[[
                                                                                   
        CCCCCCCCCCCCC                                     kkkkkkkk             iiii  
     CCC::::::::::::C                                     k::::::k            i::::i 
   CC:::::::::::::::C                                     k::::::k             iiii  
  C:::::CCCCCCCC::::C                                     k::::::k                   
 C:::::C       CCCCCC   ooooooooooo   rrrrr   rrrrrrrrr    k:::::k    kkkkkkkiiiiiii 
C:::::C               oo:::::::::::oo r::::rrr:::::::::r   k:::::k   k:::::k i:::::i 
C:::::C              o:::::::::::::::or:::::::::::::::::r  k:::::k  k:::::k   i::::i 
C:::::C              o:::::ooooo:::::orr::::::rrrrr::::::r k:::::k k:::::k    i::::i 
C:::::C              o::::o     o::::o r:::::r     r:::::r k::::::k:::::k     i::::i 
C:::::C              o::::o     o::::o r:::::r     rrrrrrr k:::::::::::k      i::::i 
C:::::C              o::::o     o::::o r:::::r             k:::::::::::k      i::::i 
 C:::::C       CCCCCCo::::o     o::::o r:::::r             k::::::k:::::k     i::::i 
  C:::::CCCCCCCC::::Co:::::ooooo:::::o r:::::r            k::::::k k:::::k   i::::::i
   CC:::::::::::::::Co:::::::::::::::o r:::::r            k::::::k  k:::::k  i::::::i
     CCC::::::::::::C oo:::::::::::oo  r:::::r            k::::::k   k:::::k i::::::i
        CCCCCCCCCCCCC   ooooooooooo    rrrrrrr            kkkkkkkk    kkkkkkkiiiiiiii
                                                                                 
--]]

class "Corki"

function Corki:__init()

	if charName ~= "Corki" then return end

	self:LoadSpells()
	self:LoadMenu()

	Callback.Add("Tick", function() self:OnTick() end)
	Callback.Add("Draw", function() self:OnDraw() end)
end

function Corki:LoadSpells()

	Q = { range = 825,	width = 250, 	speed = 1200,	delay = 0.25 }
	W = { range = 600, 	width = 160, 	speed = 700,	delay = 0.25 }
	E = { range = 600, 					speed = 902,	delay = 0.25 }
	R = { range = 1225, width = 40, 	speed = 1650, 	delay = 0.25,	collision = Collision:SetSpell(1225, 1650, 0.25, 40, true) }
end

function Corki:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Misc", name = "Misc Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Harass:MenuElement({id = "useR", name = "Use R", value = true})
		Menu.Harass:MenuElement({id = "rCount", name = "Keep X Stacks", value = 3, min = 0, max = 7, step = 1})

	--- Draw ---
		Menu.Misc:MenuElement({id = "drawMis", name = "Draw Missile Count", value = true})
		Menu.Misc:MenuElement({id = "drawBig", name = "Draw Big One", value = true})
end

function Corki:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)	
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)

	if Orb.combo and not isEvading then
		self:Combo()
	elseif Orb.harass and not isEvading then
		self:Harass()
	end
end

function Corki:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) and myHero.attackData.state ~= STATE_WINDUP then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.range then
					Utils:CastSpell(HK_Q, nPred, Q.range)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < Q.range then				
					Utils:CastSpell(HK_Q, aimPos, Q.range)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < Q.range then
						Utils:CastSpell(HK_Q, aimPos2, Q.range)
					end
				end
			end
		end
	end
end

function Corki:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range * 0.5) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Utils:CastSpell(HK_W, myHero.pos:Extended(unit.pos, W.range), W.range)
		end
	end
end

function Corki:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) then
		if myHero.attackData.state ~= STATE_WINDUP then
			Control.CastSpell(HK_E)
		end
	end
end

function Corki:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) and myHero.attackData.state ~= STATE_WINDUP then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, R.range, R.delay, R.speed, R.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, R.range, R.delay, R.speed, R.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, R.speed, R.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < R.range then
					local col, units = R.collision:__GetCollision(myHero.pos, nPred, 5)						
					if not col then
						Utils:CastSpell(HK_R, nPred, R.range, R.delay)
					end
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < R.range then
					local col, units = R.collision:__GetCollision(myHero.pos, aimPos, 5)					
					if not col then
						Utils:CastSpell(HK_R, aimPos, R.range, R.delay)
					end
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < R.range then
						local col, units = R.collision:__GetCollision(myHero.pos, aimPos2, 5)							
						if not col then
							Utils:CastSpell(HK_R, aimPos2, R.range, R.delay)
						end
					end
				end
			end
		end
	end
end

function Corki:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Corki:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Harass.useR:Value() and Utils:IsReady(_R)
	local rCount = Menu.Harass.rCount:Value()
	local rAmmo = myHero:GetSpellData(_R).ammo

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget then
		if rAmmo > rCount then
			self:CastRcombo(self.rTarget)
		end
	end
end

function Corki:OnDraw()
	
	local myHeroPos2D = myHero.pos:To2D()
	if Menu.Misc.drawMis:Value() then
		Draw.Text(myHero:GetSpellData(_R).ammo, 18, myHeroPos2D.x - 20, myHeroPos2D.y + 40, Draw.Color(255,255,0,0))
	end
	if Menu.Misc.drawBig:Value() then
		if Utils:HasBuff(myHero, "mbcheck2") then
			Draw.Text("Big One!", 12, myHeroPos2D.x - 20, myHeroPos2D.y + 60, Draw.Color(255,255,0,0))
		end
	end
end

--[[                                                                                                                                        
                                                                                                                                        
EEEEEEEEEEEEEEEEEEEEEE                                             lllllll                                                              
E::::::::::::::::::::E                                             l:::::l                                                              
E::::::::::::::::::::E                                             l:::::l                                                              
EE::::::EEEEEEEEE::::E                                             l:::::l                                                              
  E:::::E       EEEEEEvvvvvvv           vvvvvvv    eeeeeeeeeeee     l::::l yyyyyyy           yyyyyyynnnn  nnnnnnnn    nnnn  nnnnnnnn    
  E:::::E              v:::::v         v:::::v   ee::::::::::::ee   l::::l  y:::::y         y:::::y n:::nn::::::::nn  n:::nn::::::::nn  
  E::::::EEEEEEEEEE     v:::::v       v:::::v   e::::::eeeee:::::ee l::::l   y:::::y       y:::::y  n::::::::::::::nn n::::::::::::::nn 
  E:::::::::::::::E      v:::::v     v:::::v   e::::::e     e:::::e l::::l    y:::::y     y:::::y   nn:::::::::::::::nnn:::::::::::::::n
  E:::::::::::::::E       v:::::v   v:::::v    e:::::::eeeee::::::e l::::l     y:::::y   y:::::y      n:::::nnnn:::::n  n:::::nnnn:::::n
  E::::::EEEEEEEEEE        v:::::v v:::::v     e:::::::::::::::::e  l::::l      y:::::y y:::::y       n::::n    n::::n  n::::n    n::::n
  E:::::E                   v:::::v:::::v      e::::::eeeeeeeeeee   l::::l       y:::::y:::::y        n::::n    n::::n  n::::n    n::::n
  E:::::E       EEEEEE       v:::::::::v       e:::::::e            l::::l        y:::::::::y         n::::n    n::::n  n::::n    n::::n
EE::::::EEEEEEEE:::::E        v:::::::v        e::::::::e          l::::::l        y:::::::y          n::::n    n::::n  n::::n    n::::n
E::::::::::::::::::::E         v:::::v          e::::::::eeeeeeee  l::::::l         y:::::y           n::::n    n::::n  n::::n    n::::n
E::::::::::::::::::::E          v:::v            ee:::::::::::::e  l::::::l        y:::::y            n::::n    n::::n  n::::n    n::::n
EEEEEEEEEEEEEEEEEEEEEE           vvv               eeeeeeeeeeeeee  llllllll       y:::::y             nnnnnn    nnnnnn  nnnnnn    nnnnnn
                                                                                 y:::::y                                                
                                                                                y:::::y                                                 
                                                                               y:::::y                                                  
                                                                              y:::::y                                                   
                                                                             yyyyyyy                                                    
                                                                                                                                                                                                                                                                                
--]]

class "Evelynn"

function Evelynn:__init()

	if charName ~= "Evelynn" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.isQ1 = true

	Callback.Add("Tick", function() self:OnTick() end)
end

function Evelynn:LoadSpells()

	Q = { range = 800,								width = myHero:GetSpellData(_Q).width,	speed = myHero:GetSpellData(_Q).speed,	delay = 0.25,	
		collision = Collision:SetSpell(800, myHero:GetSpellData(_Q).speed, 0.25, myHero:GetSpellData(_Q).width * 1.5, true) }
	W = { range = 1200,								width = myHero:GetSpellData(_W).width,	speed = myHero:GetSpellData(_W).speed,	delay = 0.25 }
	E = { range = 300,								width = 300,							speed = myHero:GetSpellData(_E).speed,	delay = 0.25 }
	R = { range = 450,								width = myHero:GetSpellData(_R).width,	speed = myHero:GetSpellData(_R).speed,	delay = 0.25 }
end

function Evelynn:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Clear", name = "Clear Settings", type = MENU})

	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Clear ---
		Menu.Clear:MenuElement({id = "useQ", name = "Use Q", value = true})
end

function Evelynn:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)
	self.isQ1 = myHero:GetSpellData(_Q).name == "EvelynnQ"

	self:UpdateWrange()

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		elseif Orb.laneClear or Orb.jungleClear then
			self:Clear()
		end
	end
end

function Evelynn:HasWmark(unit)

	for i = 1, LocalGameParticleCount() do
		local particle = LocalGameParticle(i)
		if particle then
			if Utils:GetDistance(particle.pos, unit.pos) <= 200 then
				local name = particle.name:lower()
				if name == "evelynn_base_w_fizz_mark_decay" then
					return true
				end
			end
		end
	end
	return false	
end

function Evelynn:HasRmark(unit)

	for i = 1, LocalGameParticleCount() do
		local particle = LocalGameParticle(i)
		if particle then
			if Utils:GetDistance(particle.pos, unit.pos) <= 200 then
				local name = particle.name:lower()
				if name == "evelynn_base_r_crit_indicator" then
					return true
				end
			end
		end
	end
	return false	
end

function Evelynn:UpdateWrange()

	local wLvl = myHero:GetSpellData(_W).level
	local range = 1200
	if wLvl > 0 then
		range = ({1200, 1300, 1400, 1500, 1600})[wLvl]
		W.range = range
	end
end

function Evelynn:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) and myHero.attackData.state ~= STATE_WINDUP then
		if self.isQ1 then
			local t, aimPos = HPred:GetReliableTarget(myHero.pos, Q.range, Q.delay, Q.speed, Q.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, Q.range, Q.delay, Q.speed, Q.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, Q.speed, Q.delay)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			if useNpred then
				if nPred then
					if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) < Q.range then
						local col, units = Q.collision:__GetCollision(myHero.pos, nPred, 5)						
						if not col then
							Utils:CastSpell(HK_Q, nPred, Q.range, Q.delay)
						end
					end					
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) < Q.range then
						local col, units = Q.collision:__GetCollision(myHero.pos, aimPos, 5)					
						if not col then
							Utils:CastSpell(HK_Q, aimPos, Q.range, Q.delay)
						end
					end
				end
				if hitChance and aimPos2 then
					if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
						if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) < Q.range then
							local col, units = Q.collision:__GetCollision(myHero.pos, aimPos2, 5)							
							if not col then
								Utils:CastSpell(HK_Q, aimPos2, Q.range, Q.delay)
							end
						end
					end
				end
			end
		else
			Control.CastSpell(HK_Q)
		end
	end
end

function Evelynn:CastWcombo(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) and myHero.attackData.state ~= STATE_WINDUP then
		if unit.pos:To2D().onScreen then
			Utils:CastSpell(HK_W, unit.pos, W.range)
		end
	end
end

function Evelynn:CastEcombo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) and myHero.attackData.state ~= STATE_WINDUP then
		if unit.pos:To2D().onScreen then
			Utils:CastSpell(HK_E, unit.pos, E.range)
		end
	end
end

function Evelynn:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) and myHero.attackData.state ~= STATE_WINDUP then
		if unit.pos:To2D().onScreen and self:GetRdmg(unit) > unit.health then
			Utils:CastSpell(HK_R, unit.pos, R.range)
		end
	end
end

function Evelynn:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Evelynn:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		self:CastEcombo(self.eTarget)
	end
end

function Evelynn:Clear()

	local qReady = Menu.Clear.useQ:Value() and Utils:IsReady(_Q)

	if qReady then
		for i = 1, LocalGameMinionCount() do
			local m = LocalGameMinion(i)
			if m and Utils:IsValid(m, myHero.pos, Q.range) then
				if Utils:GetDistance(m.pos, myHero.pos) <= Q.range then
					Utils:CastSpell(HK_Q, m.pos, Q.range)
				end
			end
		end
	end
end

function Evelynn:GetRdmg(unit)
	
	local total = 0
	local rLvl = myHero:GetSpellData(_R).level
	if rLvl > 0 then
		local raw = ({ 150, 275, 400 })[rLvl]
		local ap = myHero.ap * .75

		if self:HasRmark(unit) then
			total = raw * 2 + ap * 2
		else
			total = raw + ap
		end
	end
	return Utils:CalculateMagicalDamage(unit, total)
end

--[[                                                                                
                                                                                    
IIIIIIIIII                                        lllllll   iiii                    
I::::::::I                                        l:::::l  i::::i                   
I::::::::I                                        l:::::l   iiii                    
II::::::II                                        l:::::l                           
  I::::I  rrrrr   rrrrrrrrr       eeeeeeeeeeee     l::::l iiiiiii   aaaaaaaaaaaaa   
  I::::I  r::::rrr:::::::::r    ee::::::::::::ee   l::::l i:::::i   a::::::::::::a  
  I::::I  r:::::::::::::::::r  e::::::eeeee:::::ee l::::l  i::::i   aaaaaaaaa:::::a 
  I::::I  rr::::::rrrrr::::::re::::::e     e:::::e l::::l  i::::i            a::::a 
  I::::I   r:::::r     r:::::re:::::::eeeee::::::e l::::l  i::::i     aaaaaaa:::::a 
  I::::I   r:::::r     rrrrrrre:::::::::::::::::e  l::::l  i::::i   aa::::::::::::a 
  I::::I   r:::::r            e::::::eeeeeeeeeee   l::::l  i::::i  a::::aaaa::::::a 
  I::::I   r:::::r            e:::::::e            l::::l  i::::i a::::a    a:::::a 
II::::::II r:::::r            e::::::::e          l::::::li::::::ia::::a    a:::::a 
I::::::::I r:::::r             e::::::::eeeeeeee  l::::::li::::::ia:::::aaaa::::::a 
I::::::::I r:::::r              ee:::::::::::::e  l::::::li::::::i a::::::::::aa:::a
IIIIIIIIII rrrrrrr                eeeeeeeeeeeeee  lllllllliiiiiiii  aaaaaaaaaa  aaaa
                                                                                  
--]]

class "Irelia"

function Irelia:__init()

	if charName ~= "Irelia" then return end

	self:LoadSpells()
	self:LoadMenu()
	self.lastE = GetTickCount()
	self.lastBuff = GetTickCount()
	self.chargingW = false
	self.wTime = GetTickCount()
	self.lastW = GetTickCount()

	Callback.Add("Tick", function() self:OnTick() end)
end

function Irelia:LoadSpells()

	Q = { range = 625,	delay = 0.25 }
	W = { range = 600, 	delay = 0.25,	width = 100,	speed = _huge }
	E = { range = 925,  delay = 0.25,	width = 50,		speed = 1800 }
	R = { range = 1050, delay = 0.25,	width = 160,	speed = 2000 }
end

function Irelia:LoadMenu()

	Menu:MenuElement({id = "Combo", name = "Combo Settings", type = MENU})
	Menu:MenuElement({id = "Harass", name = "Harass Settings", type = MENU})
	Menu:MenuElement({id = "Clear", name = "Clear Settings", type = MENU})
	--- Combo ---
		Menu.Combo:MenuElement({id = "useQ", name = "Use Q", value = true})
		Menu.Combo:MenuElement({id = "useQj", name = "Use Q gapclose", value = true})
		Menu.Combo:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Combo:MenuElement({id = "useE", name = "Use E", value = true})
		Menu.Combo:MenuElement({id = "useR", name = "Use R", value = true})

	--- Harass ---
		Menu.Harass:MenuElement({id = "useQ", name = "Use Q", value = false})
		Menu.Harass:MenuElement({id = "useW", name = "Use W", value = true})
		Menu.Harass:MenuElement({id = "useE", name = "Use E", value = true})

	--- Clear ---

		Menu.Clear:MenuElement({id = "useQ", name = "Use Q", value = true})
end

function Irelia:OnTick()

	self.qTarget = Utils:GetTarget(Q.range)
	self.wTarget = Utils:GetTarget(W.range)
	self.eTarget = Utils:GetTarget(E.range)
	self.rTarget = Utils:GetTarget(R.range)
	self.gCtarget = Utils:GetTarget(1200)
	self.eName = myHero:GetSpellData(_E).name
	self:UpdateWstate()
	self:DisableOrb()

	if not isEvading then
		if Orb.combo then
			self:Combo()
		elseif Orb.harass then
			self:Harass()
		elseif Orb.jungleClear or Orb.laneClear then
			self:Clear()
		end
	end
end

function Irelia:DisableOrb()

	local hasWbuff = Utils:HasBuff(myHero, "ireliawdefense")

	if hasWbuff then
		Orb:DisableMovement(true)
		Orb:DisableAttacks(true)
	else
		Orb:DisableMovement(false)
		Orb:DisableAttacks(false)
	end
end

function Irelia:HasMark(unit)
	
	if self.lastBuff + 100 <= GetTickCount() then
		for i = 1, unit.buffCount do
			local buff = unit:GetBuff(i)
			if buff and buff.count > 0 and buff.name == "ireliamark" then
				self.lastBuff = GetTickCount()
				return true
			end
		end
		self.lastBuff = GetTickCount()
		return false
	end
end

function Irelia:GetE1()

	if self.lastE + 100 <= GetTickCount() then
		for i = 1, LocalGameObjectCount() do
			local obj = LocalGameObject(i)
			if obj then
				if obj.name == "Irelia_Base_E_Team_Indicator" then
					self.lastE = GetTickCount()
					return obj
				end
			end
		end
		self.lastE = GetTickCount()
	end
end

function Irelia:CastQcombo(unit)

	if Utils:IsValid(unit, myHero.pos, Q.range) and myHero.attackData.state ~= STATE_WINDUP then
		if self:HasMark(unit) or self:GetQdmg(unit) > unit.health then
			Utils:CastSpell(HK_Q, unit.pos, Q.range)
		end
	end
end

function Irelia:CastWcombo()

	for i = 1, #Utils:GetEnemyHeroesInRange(myHero.pos, W.range * .9) do
		local current = Utils:GetEnemyHeroesInRange(myHero.pos, W.range * .9)[i]
		if current then
			if current.activeSpell and current.activeSpell.valid and
				(current.activeSpell.target == myHero.handle or 
					Utils:GetDistance(current.activeSpell.placementPos, myHero.pos) <= myHero.boundingRadius * 2 + current.activeSpell.width) then
				local startPos = current.activeSpell.startPos
				local placementPos = current.activeSpell.placementPos
				local width = 0
				if current.activeSpell.width > 0 then
					width = current.activeSpell.width
				else
					width = 100
				end
				local distance = Utils:GetDistance(myHero.pos, placementPos)											
				if current.activeSpell.target == myHero.handle then
					if not self.chargingW then
						self:CastWstart(current)
					else
						self:CastWcharged(current)
					end
					return
				else
					if distance <= width * 2 + myHero.boundingRadius then
						if not self.chargingW then
							self:CastWstart(current)
						else
							self:CastWcharged(current)
						end
						break						
					end
				end
			end
		end
	end
end

function Irelia:CastWstart(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) and myHero:GetSpellData(_E).name ~= "IreliaE2" then
		if not self.chargingW then
			local nPred = Utils:GetPred(unit, W.speed, W.delay)
			if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) <= W.range then
				Control.KeyDown(HK_W)
				self.lastW = GetTickCount()
			end
		end
	end
end

function Irelia:CastWcharged(unit)

	if Utils:IsValid(unit, myHero.pos, W.range) and GetTickCount() > self.lastW + 1000 and GetTickCount() < self.lastW + 1750 then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, W.range, W.delay, W.speed, W.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, W.range, W.delay, W.speed, W.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, W.speed, W.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) <= W.range then
					Control.SetCursorPos(nPred)
				end
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) <= W.range then
					Control.SetCursorPos(aimPos)
				end
			end
			if hitChance and aimPos2 then
				if (hitChance >= minAccuracy) then --or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) <= W.range then
						Control.SetCursorPos(aimPos2)
					end
				end
			end
		end
	end
end

function Irelia:UpdateWstate()

	self.wTime = GetTickCount() - self.lastW

	local hasWbuff = Utils:HasBuff(myHero, "ireliawdefense")

	if hasWbuff and not self.chargingW then
		self.chargingW = true
	end
	if self.chargingW and not hasWbuff then
		DelayAction(function()
			if self.chargingW and not hasWbuff then
				self.chargingW = false
			end
		end,0.25)
	end

	if Control.IsKeyDown(HK_W) == true and not self.chargingW then
		DelayAction(function()
			if Control.IsKeyDown(HK_W) == true and not self.chargingW then
				Control.KeyUp(HK_W)
			end
		end,0.25)
	end

	if Control.IsKeyDown(HK_W) == true and #Utils:GetEnemyHeroesInRange(myHero.pos, W.range * .9) == 0 then
		DelayAction(function()
			if Control.IsKeyDown(HK_W) == true and #Utils:GetEnemyHeroesInRange(myHero.pos, W.range * .9) == 0 then
				Control.KeyUp(HK_W)
			end
		end,0.25)
	end

end
function Irelia:CastE1combo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) and myHero:GetSpellData(_E).name == "IreliaE" then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, E.speed, E.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		local eRange = E.range * 0.33
		if useNpred then
			if nPred then
				local extendedPos = myHero.pos:Extended(nPred, eRange + Utils:GetDistance(nPred, myHero.pos))
				if extendedPos:To2D().onScreen and Utils:GetDistance(extendedPos, myHero.pos) <= E.range then
					Utils:CastSpell(HK_E, extendedPos, E.range, E.delay)
				end					
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				local extendedPos = myHero.pos:Extended(aimPos, eRange + Utils:GetDistance(aimPos, myHero.pos))
				if extendedPos:To2D().onScreen and Utils:GetDistance(extendedPos, myHero.pos) <= E.range then
					Utils:CastSpell(HK_E, extendedPos, E.range, E.delay)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					local extendedPos = myHero.pos:Extended(aimPos2, eRange + Utils:GetDistance(aimPos2, myHero.pos))			
					if extendedPos:To2D().onScreen and Utils:GetDistance(extendedPos, myHero.pos) <= E.range then
						Utils:CastSpell(HK_E, extendedPos, E.range, E.delay)
					end
				end
			end
		end
	end
end

function Irelia:CastE2combo(unit)

	if Utils:IsValid(unit, myHero.pos, E.range) and myHero:GetSpellData(_E).name == "IreliaE2" then
		local bladeObj = self:GetE1()
		if bladeObj then
			local t, aimPos = HPred:GetReliableTarget(bladeObj.pos, E.range, E.delay, E.speed, E.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
			local hitChance, aimPos2 = HPred:GetHitchance(bladeObj.pos, unit, E.range, E.delay, E.speed, E.width, false, unit.charName)
			local useNpred = Menu.AiO.Pred.pred:Value() == 1
			local nPred = Utils:GetPred(unit, E.speed, E.delay, bladeObj.pos)
			local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
			local objPos = bladeObj.pos
			local objPos2D = bladeObj.pos:To2D()
			local unitPos = unit.pos
			local eRange = E.range * 0.33
			if useNpred then
				if nPred then
					local extendedPos = objPos:Extended(nPred, Utils:GetDistance(objPos, unitPos) + eRange)
					if extendedPos:To2D().onScreen and Utils:GetDistance(extendedPos, myHero.pos) <= E.range then
						Utils:CastSpell(HK_E, extendedPos, E.range)
					end					
				end
			else
				if t and aimPos and t.networkID == unit.networkID then
					local extendedPos = objPos:Extended(aimPos, Utils:GetDistance(objPos, unitPos) + eRange)
					if extendedPos:To2D().onScreen and Utils:GetDistance(extendedPos, myHero.pos) <= E.range then
						Utils:CastSpell(HK_E, extendedPos, E.range, E.delay)
					end
				end
				if hitChance and aimPos2 then
					local extendedPos = objPos:Extended(aimPos2, Utils:GetDistance(objPos, unitPos) + eRange)
					if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
						if extendedPos:To2D().onScreen and Utils:GetDistance(extendedPos, myHero.pos) < E.range then
							Utils:CastSpell(HK_E, extendedPos, E.range, E.delay)
						end
					end
				end
			end
		end
	end
end

function Irelia:CastRcombo(unit)

	if Utils:IsValid(unit, myHero.pos, R.range) and self:GetComboDmg(unit) > unit.health and unit.health > self:GetQdmg(unit) then
		local t, aimPos = HPred:GetReliableTarget(myHero.pos, R.range, R.delay, R.speed, R.width, Menu.AiO.Pred.HPred.reactionTime:Value(), false)
		local hitChance, aimPos2 = HPred:GetHitchance(myHero.pos, unit, R.range, R.delay, R.speed, R.width, false, unit.charName)
		local useNpred = Menu.AiO.Pred.pred:Value() == 1
		local nPred = Utils:GetPred(unit, R.speed, R.delay)
		local minAccuracy = Menu.AiO.Pred.HPred.accuracy:Value()
		if useNpred then
			if nPred then
				if nPred:To2D().onScreen and Utils:GetDistance(nPred, myHero.pos) <= R.range then
					Utils:CastSpell(HK_R, nPred, R.range, R.delay)
				end
			end
		else
			if t and aimPos and t.networkID == unit.networkID then
				if aimPos:To2D().onScreen and Utils:GetDistance(aimPos, myHero.pos) <= R.range then
					Utils:CastSpell(HK_R, aimPos, R.range, R.delay)
				end
			end
			if hitChance and aimPos2 then
				if hitChance >= minAccuracy then -- or (hitChance >= 1 and unit.health < self:GetComboDmg(unit)) then
					if aimPos2:To2D().onScreen and Utils:GetDistance(aimPos2, myHero.pos) <= R.range then
						Utils:CastSpell(HK_R, aimPos2, R.range, R.delay)
					end
				end
			end
		end
	end
end

function Irelia:Combo()

	local qReady = Menu.Combo.useQ:Value() and Utils:IsReady(_Q)
	local qJready = Menu.Combo.useQj:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Combo.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Combo.useE:Value() and Utils:IsReady(_E)
	local rReady = Menu.Combo.useR:Value() and Utils:IsReady(_R)
	local eName = myHero:GetSpellData(_E).name

	if qReady then

		if self.qTarget then
			self:CastQcombo(self.qTarget)
		end

		if qJready and self.gCtarget then
			if #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) > 0 then
				for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) do
				local minion = Utils:GetEnemyMinionsInRange(myHero.pos, Q.range)[i]
					if minion then
						local mPos = minion.pos
						if qReady then
							if Utils:IsValid(minion, myHero.pos, Q.range) and 
								Utils:GetDistance(mPos, self.gCtarget.pos) < Utils:GetDistance(myHero.pos, self.gCtarget.pos) then
								if minion.health < self:GetQdmg(minion) * 1.7 then
									Control.CastSpell(HK_Q, minion)
								end
							end
						end
					end
				end
			end
		end
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		if eName == "IreliaE" then
			self:CastE1combo(self.eTarget)
		else
			self:CastE2combo(self.eTarget)
		end
	end

	if rReady and self.rTarget then
		self:CastRcombo(self.rTarget)
	end
end

function Irelia:Harass()

	local qReady = Menu.Harass.useQ:Value() and Utils:IsReady(_Q)
	local wReady = Menu.Harass.useW:Value() and Utils:IsReady(_W)
	local eReady = Menu.Harass.useE:Value() and Utils:IsReady(_E)
	local eName = myHero:GetSpellData(_E).name

	if qReady and self.qTarget then
		self:CastQcombo(self.qTarget)
	end

	if wReady and self.wTarget then
		self:CastWcombo(self.wTarget)
	end

	if eReady and self.eTarget then
		if eName == "IreliaE" then
			self:CastE1combo(self.eTarget)
		else
			self:CastE2combo(self.eTarget)
		end
	end
end

function Irelia:Clear()
	
	local qReady = Menu.Clear.useQ:Value() and Utils:IsReady(_Q)

	if qReady then
		if #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) > 0 then
			for i = 1, #Utils:GetEnemyMinionsInRange(myHero.pos, Q.range) do
				local minion = Utils:GetEnemyMinionsInRange(myHero.pos, Q.range)[i]
				if minion then
					local mPos = minion.pos			
					if Utils:GetDistance(mPos, mousePos) < 200 then
						if minion.health < self:GetQdmg(minion) * 1.7 then
							Control.CastSpell(HK_Q, minion)
						end
					end
				end
			end
		end
	end
end

function Irelia:GetQdmg(unit)
	
	if not unit then return 0 end
	local total = 0
	local qLvl = myHero:GetSpellData(_Q).level
	if qLvl > 0 then
		local raw = ({ 10, 30, 50, 70, 90 })[qLvl]
		local bonusDmg = myHero.totalDamage * .7
		total = (raw + bonusDmg)
	end
	return Utils:CalculatePhysicalDamage(unit, total)
end

function Irelia:GetEdmg(unit)

	if not unit then return 0 end
	local total = 0
	local eLvl = myHero:GetSpellData(_E).level
	if eLvl > 0 then
		local raw = ({ 80, 120, 160, 200, 240 })[eLvl]
		local bonusDmg = myHero.ap * .8
		total = (raw + bonusDmg)
	end
	return Utils:CalculateMagicalDamage(unit, total)
end

function Irelia:GetRdmg(unit)

	if not unit then return 0 end
	local total = 0
	local rLvl = myHero:GetSpellData(_R).level
	if rLvl > 0 then
		local raw1 = ({ 125, 225, 325 })[rLvl]
		local raw2 =  ({ 75, 125, 175 })[rLvl]
		local bonusDmg = myHero.ap * .7
		total = (raw1 + raw2 + bonusDmg * 2)
	end
	return Utils:CalculatePhysicalDamage(unit, total)
end

function Irelia:GetComboDmg(unit)

	local totalDmg = 0

	if unit then
		local qLvl = myHero:GetSpellData(_Q).level
		local eLvl = myHero:GetSpellData(_E).level
		local rLvl = myHero:GetSpellData(_R).level
		if qLvl > 0 then
			totalDmg = totalDmg + self:GetQdmg(unit)
		end
		if eLvl > 0 and Utils:IsReady(_E) then
			totalDmg = totalDmg + self:GetEdmg(unit) + self:GetQdmg(unit)
		end
		if rLvl > 0 and Utils:IsReady(_R) then
			totalDmg = totalDmg + self:GetRdmg(unit) + self:GetQdmg(unit)
		end
	end
	return totalDmg
end

--[[
        (    (      (                                 
        )\ ) )\ )   )\ )                  )  (        
    (  (()/((()/(  (()/(   (      )    ( /(  )\   (   
    )\  /(_))/(_))  /(_)) ))\    (     )\())((_) ))\  
 _ ((_)(_)) (_))_| (_))  /((_)   )\  '((_)\  _  /((_) 
| | | || _ \| |_   | _ \(_))(  _((_)) | |(_)| |(_))   
| |_| ||   /| __|  |   /| || || '  \()| '_ \| |/ -_)  
 \___/ |_|_\|_|    |_|_\ \_,_||_|_|_| |_.__/|_|\___|  
 (       (                                            
 )\      )\                                           
((_) (  ((_)                                          
 _   )\  _                                            
| | ((_)| |                                           
| |/ _ \| |                                           
|_|\___/|_|                                           
                                                      
--]]

class "Rumble"

function Rumble:__init()

	if charName ~= "Rumble" then return end
	Callback.Add("Tick", function() self:OnTick() end)
end
function Rumble:OnTick()

	if Orb.combo then
		self:Combo()
	end
end

function Rumble:Combo()

	if Utils:IsReady(_Q) then
		Control.CastSpell(HK_Q)
	end

	if Utils:IsReady(_W) then
		Control.CastSpell(HK_W)
	end
end

--[[                                                                                                       
                                                                                               dddddddd
HHHHHHHHH     HHHHHHHHHPPPPPPPPPPPPPPPPP                                                       d::::::d
H:::::::H     H:::::::HP::::::::::::::::P                                                      d::::::d
H:::::::H     H:::::::HP::::::PPPPPP:::::P                                                     d::::::d
HH::::::H     H::::::HHPP:::::P     P:::::P                                                    d:::::d 
  H:::::H     H:::::H    P::::P     P:::::Prrrrr   rrrrrrrrr       eeeeeeeeeeee        ddddddddd:::::d 
  H:::::H     H:::::H    P::::P     P:::::Pr::::rrr:::::::::r    ee::::::::::::ee    dd::::::::::::::d 
  H::::::HHHHH::::::H    P::::PPPPPP:::::P r:::::::::::::::::r  e::::::eeeee:::::ee d::::::::::::::::d 
  H:::::::::::::::::H    P:::::::::::::PP  rr::::::rrrrr::::::re::::::e     e:::::ed:::::::ddddd:::::d 
  H:::::::::::::::::H    P::::PPPPPPPPP     r:::::r     r:::::re:::::::eeeee::::::ed::::::d    d:::::d 
  H::::::HHHHH::::::H    P::::P             r:::::r     rrrrrrre:::::::::::::::::e d:::::d     d:::::d 
  H:::::H     H:::::H    P::::P             r:::::r            e::::::eeeeeeeeeee  d:::::d     d:::::d 
  H:::::H     H:::::H    P::::P             r:::::r            e:::::::e           d:::::d     d:::::d 
HH::::::H     H::::::HHPP::::::PP           r:::::r            e::::::::e          d::::::ddddd::::::dd
H:::::::H     H:::::::HP::::::::P           r:::::r             e::::::::eeeeeeee   d:::::::::::::::::d
H:::::::H     H:::::::HP::::::::P           r:::::r              ee:::::::::::::e    d:::::::::ddd::::d
HHHHHHHHH     HHHHHHHHHPPPPPPPPPP           rrrrrrr                eeeeeeeeeeeeee     ddddddddd   ddddd                                                                                                    
                                                                                                       
--]]
class "HPred"
	
local _reviveQueryFrequency = .2
local _lastReviveQuery = LocalGameTimer()
local _reviveLookupTable = 
	{ 
		["LifeAura.troy"] = 4, 
		["ZileanBase_R_Buf.troy"] = 3,
		["Aatrox_Base_Passive_Death_Activate"] = 3
		
		--TwistedFate_Base_R_Gatemarker_Red
			--String match would be ideal.... could be different in other skins
	}

--Stores a collection of spells that will cause a character to blink
	--Ground targeted spells go towards mouse castPos with a maximum range
	--Hero/Minion targeted spells have a direction type to determine where we will land relative to our target (in front of, behind, etc)
	
--Key = Spell name
--Value = range a spell can travel, OR a targeted end position type, OR a list of particles the spell can teleport to	
local _blinkSpellLookupTable = 
	{ 
		["EzrealArcaneShift"] = 475, 
		["RiftWalk"] = 500,
		
		--Ekko and other similar blinks end up between their start pos and target pos (in front of their target relatively speaking)
		["EkkoEAttack"] = 0,
		["AlphaStrike"] = 0,
		
		--Katarina E ends on the side of her target closest to where her mouse was... 
		--["KatarinaE"] = -255, -- Had a few crashes with Katarina in Games, also seems to be fairly inaccurate - disabled for now
		
		--Katarina can target a dagger to teleport directly to it: Each skin has a different particle name. This should cover all of them.
		--["KatarinaEDagger"] = { "Katarina_Base_Dagger_Ground_Indicator","Katarina_Skin01_Dagger_Ground_Indicator","Katarina_Skin02_Dagger_Ground_Indicator","Katarina_Skin03_Dagger_Ground_Indicator","Katarina_Skin04_Dagger_Ground_Indicator","Katarina_Skin05_Dagger_Ground_Indicator","Katarina_Skin06_Dagger_Ground_Indicator","Katarina_Skin07_Dagger_Ground_Indicator" ,"Katarina_Skin08_Dagger_Ground_Indicator","Katarina_Skin09_Dagger_Ground_Indicator"  }, 
	}

local _blinkLookupTable = 
	{ 
		"global_ss_flash_02.troy",
		"Lissandra_Base_E_Arrival.troy",
		"Leblanc_Base_W_return_activation.troy"
		--TODO: Check if liss/Leblanc have diff skill versions. MOST likely dont but worth checking for completion sake
		
		--Zed uses 'switch shadows'... It will require some special checks to choose the shadow he's going TO not from...
		--Shaco deceive no longer has any particles where you jump to so it cant be tracked (no spell data or particles showing path)
		
	}

local _cachedRevives = {}
local _cachedTeleports = {}
local _movementHistory = {}

--Cache of all TARGETED missiles currently running
local _cachedMissiles = {}
local _incomingDamage = {}

--Cache of active enemy windwalls so we can calculate it when dealing with collision checks
local _windwall
local _windwallStartPos
local _windwallWidth

function HPred:Tick()
	--Update missile cache
	--DISABLED UNTIL LATER.
	--self:CacheMissiles()
	
	--self:CacheParticles()
	
	--Check for revives and record them	
	if LocalGameTimer() - _lastReviveQuery < _reviveQueryFrequency then return end
	_lastReviveQuery=LocalGameTimer()
	
	--Remove old cached revives
	for _, revive in pairs(_cachedRevives) do
		if LocalGameTimer() > revive.expireTime + .5 then
			_cachedRevives[_] = nil
		end
	end
	
	--Cache new revives
	for i = 1, LocalGameParticleCount() do 
		local particle = LocalGameParticle(i)
		if particle and not _cachedRevives[particle.networkID] and  _reviveLookupTable[particle.name] then
			_cachedRevives[particle.networkID] = {}
			_cachedRevives[particle.networkID]["expireTime"] = LocalGameTimer() + _reviveLookupTable[particle.name]			
			local target = self:GetHeroByPosition(particle.pos)
			if target and target.isEnemy then				
				_cachedRevives[particle.networkID]["target"] = target
				_cachedRevives[particle.networkID]["pos"] = target.pos
				_cachedRevives[particle.networkID]["isEnemy"] = target.isEnemy	
			end
		end
	end
	
	--Update hero movement history	
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t then
			self:UpdateMovementHistory(t)
		end
	end
	
	--Remove old cached teleports	
	for _, teleport in pairs(_cachedTeleports) do
		if teleport and LocalGameTimer() > teleport.expireTime + .5 then
			_cachedTeleports[_] = nil
		end
	end	
	--Update teleport cache
	self:CacheTeleports()
end

function HPred:GetEnemyNexusPosition()
	--This is slightly wrong. It represents fountain not the nexus. Fix later.
	if myHero.team == 100 then return Vector(14340, 171.977722167969, 14390); else return Vector(396,182.132507324219,462); end
end


function HPred:GetGuarenteedTarget(source, range, delay, speed, radius, timingAccuracy, checkCollision)
	--Get hourglass enemies
	target, aimPosition =self:GetHourglassTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get reviving target
	target, aimPosition =self:GetRevivingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end	
	
	--Get teleporting enemies
	target, aimPosition =self:GetTeleportingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)	
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get stunned enemies
	local target, aimPosition =self:GetImmobileTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end
end


function HPred:GetReliableTarget(source, range, delay, speed, radius, timingAccuracy, checkCollision)
	--TODO: Target whitelist. This will target anyone which is definitely not what we want
	--For now we can handle in the champ script. That will cause issues with multiple people in range who are goood targets though.
	
	
	--Get hourglass enemies
	target, aimPosition =self:GetHourglassTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get reviving target
	target, aimPosition =self:GetRevivingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get channeling enemies
	target, aimPosition =self:GetChannelingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
		if target and aimPosition then
		return target, aimPosition
	end
	
	--Get teleporting enemies
	target, aimPosition =self:GetTeleportingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)	
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get instant dash enemies
	target, aimPosition =self:GetInstantDashTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end	
	
	--Get dashing enemies
	target, aimPosition =self:GetDashingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius, midDash)
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get stunned enemies
	local target, aimPosition =self:GetImmobileTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	if target and aimPosition then
		return target, aimPosition
	end
	
	--Get blink targets
	--target, aimPosition =self:GetBlinkTarget(source, range, speed, delay, checkCollision, radius)
	--if target and aimPosition then
	--	return target, aimPosition
	--end	
end

--Will return how many allies or enemies will be hit by a linear spell based on current waypoint data.
function HPred:GetLineTargetCount(source, aimPos, delay, speed, width, targetAllies)
	local targetCount = 0
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t and self:CanTargetALL(t) and (targetAllies or t.isEnemy) then
			local predictedPos = self:PredictUnitPosition(t, delay+ self:GetDistance(source, t.pos) / speed)
			local proj1, pointLine, isOnSegment = self:VectorPointProjectionOnLineSegment(source, aimPos, predictedPos)
			if proj1 and isOnSegment and (self:GetDistanceSqr(predictedPos, proj1) <= (t.boundingRadius + width) * (t.boundingRadius + width)) then
				targetCount = targetCount + 1
			end
		end
	end
	return targetCount
end

--Will return the valid target who has the highest hit chance and meets all conditions (minHitChance, whitelist check, etc)
function HPred:GetUnreliableTarget(source, range, delay, speed, radius, checkCollision, minimumHitChance, whitelist)
	local _validTargets = {}
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t and self:CanTarget(t) and (not whitelist or whitelist[t.charName]) then			
			local hitChance, aimPosition = self:GetHitchance(source, t, range, delay, speed, radius, checkCollision)		
			if hitChance >= minimumHitChance then
				_validTargets[t.charName] = {["hitChance"] = hitChance, ["aimPosition"] = aimPosition}
			end
		end
	end
	
	local rHitChance = 0
	local rAimPosition
	for targetName, targetData in pairs(_validTargets) do
		if targetData.hitChance > rHitChance then
			rHitChance = targetData.hitChance
			rAimPosition = targetData.aimPosition
		end		
	end
	
	if rHitChance >= minimumHitChance then
		return rHitChance, rAimPosition
	end	
end

function HPred:GetHitchance(source, target, range, delay, speed, radius, checkCollision)	
	local hitChance = 1	
	
	local aimPosition = self:PredictUnitPosition(target, delay + self:GetDistance(source, target.pos) / speed)	
	local interceptTime = self:GetSpellInterceptTime(source, aimPosition, delay, speed)
	local reactionTime = self:PredictReactionTime(target, .1)
	
	--If they just now changed their path then assume they will keep it for at least a short while... slightly higher chance
	if _movementHistory and _movementHistory[target.charName] and LocalGameTimer() - _movementHistory[target.charName]["ChangedAt"] < .25 then
		hitChance = 2
	end

	--If they are standing still give a higher accuracy because they have to take actions to react to it
	if not target.pathing or not target.pathing.hasMovePath then
		hitChance = 2
	end	
	
	
	local origin,movementRadius = self:UnitMovementBounds(target, interceptTime, reactionTime)
	--Our spell is so wide or the target so slow or their reaction time is such that the spell will be nearly impossible to avoid
	if movementRadius - target.boundingRadius <= radius /2 then
		origin,movementRadius = self:UnitMovementBounds(target, interceptTime, 0)
		if movementRadius - target.boundingRadius <= radius /2 then
			hitChance = 4
		else		
			hitChance = 3
		end
	end	
	
	--If they are casting a spell then the accuracy will be fairly high. if the windup is longer than our delay then it's quite likely to hit. 
	--Ideally we would predict where they will go AFTER the spell finishes but that's beyond the scope of this prediction
	if target.activeSpell and target.activeSpell.valid then
		if target.activeSpell.startTime + target.activeSpell.windup - LocalGameTimer() >= delay then
			hitChance = 5
		else			
			hitChance = 3
		end
	end
	
	--Check for out of range
	if not self:IsInRange(myHero.pos, aimPosition, range) then
		hitChance = -1
	end
	
	--Check minion block
	if hitChance > 0 and checkCollision then
		if self:IsWindwallBlocking(source, aimPosition) then
			hitChance = -1		
		elseif self:CheckMinionCollision(source, aimPosition, delay, speed, radius) then
			hitChance = -1
		end
	end
	
	return hitChance, aimPosition
end

function HPred:PredictReactionTime(unit, minimumReactionTime)
	local reactionTime = minimumReactionTime
	if not unit or not reactionTime then return end
	--If the target is auto attacking increase their reaction time by .15s - If using a skill use the remaining windup time
	if unit.activeSpell and unit.activeSpell.valid then
		local windupRemaining = unit.activeSpell.startTime + unit.activeSpell.windup - LocalGameTimer()
		if windupRemaining > 0 then
			reactionTime = windupRemaining
		end
	end
	
	return reactionTime
end

function HPred:GetDashingTarget(source, range, delay, speed, dashThreshold, checkCollision, radius, midDash)

	local target
	local aimPosition
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t and t.isEnemy and t.pathing.hasMovePath and t.pathing.isDashing and t.pathing.dashSpeed>500  then
			local dashEndPosition = t:GetPath(1)
			if self:IsInRange(source, dashEndPosition, range) then				
				--The dash ends within range of our skill. We now need to find if our spell can connect with them very close to the time their dash will end
				local dashTimeRemaining = self:GetDistance(t.pos, dashEndPosition) / t.pathing.dashSpeed
				local skillInterceptTime = self:GetSpellInterceptTime(myHero.pos, dashEndPosition, delay, speed)
				local deltaInterceptTime =skillInterceptTime - dashTimeRemaining
				if deltaInterceptTime > 0 and deltaInterceptTime < dashThreshold and (not checkCollision or not self:CheckMinionCollision(source, dashEndPosition, delay, speed, radius)) then
					target = t
					aimPosition = dashEndPosition
					return target, aimPosition
				end
			end			
		end
	end
end

function HPred:GetHourglassTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	local target
	local aimPosition
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t and t.isEnemy then		
			local success, timeRemaining = self:HasBuff(t, "zhonyasringshield")
			if success then
				local spellInterceptTime = self:GetSpellInterceptTime(myHero.pos, t.pos, delay, speed)
				local deltaInterceptTime = spellInterceptTime - timeRemaining
				if spellInterceptTime > timeRemaining and deltaInterceptTime < timingAccuracy and (not checkCollision or not self:CheckMinionCollision(source, interceptPosition, delay, speed, radius)) then
					target = t
					aimPosition = t.pos
					return target, aimPosition
				end
			end
		end
	end
end

function HPred:GetRevivingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	local target
	local aimPosition
	for _, revive in pairs(_cachedRevives) do
		if revive and revive.isEnemy then
			local interceptTime = self:GetSpellInterceptTime(source, revive.pos, delay, speed)
			if interceptTime > revive.expireTime - LocalGameTimer() and interceptTime - revive.expireTime - LocalGameTimer() < timingAccuracy then
				target = revive.target
				aimPosition = revive.pos
				return target, aimPosition
			end
		end
	end	
end

function HPred:GetInstantDashTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	local target
	local aimPosition
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t and t.isEnemy and t.activeSpell and t.activeSpell.valid and _blinkSpellLookupTable[t.activeSpell.name] then
			local windupRemaining = t.activeSpell.startTime + t.activeSpell.windup - LocalGameTimer()
			if windupRemaining > 0 then
				local endPos
				local blinkRange = _blinkSpellLookupTable[t.activeSpell.name]
				if type(blinkRange) == "table" then
					--Find the nearest matching particle to our mouse
					--local target, distance = self:GetNearestParticleByNames(t.pos, blinkRange)
					--if target and distance < 250 then					
					--	endPos = target.pos		
					--end
				elseif blinkRange > 0 then
					endPos = Vector(t.activeSpell.placementPos.x, t.activeSpell.placementPos.y, t.activeSpell.placementPos.z)					
					endPos = t.activeSpell.startPos + (endPos- t.activeSpell.startPos):Normalized() * _min(self:GetDistance(t.activeSpell.startPos,endPos), range)
				else
					local blinkTarget = self:GetObjectByHandle(t.activeSpell.target)
					if blinkTarget then				
						local offsetDirection						
						
						--We will land in front of our target relative to our starting position
						if blinkRange == 0 then				

							if t.activeSpell.name ==  "AlphaStrike" then
								windupRemaining = windupRemaining + .75
								--TODO: Boost the windup time by the number of targets alpha will hit. Need to calculate the exact times this is just rough testing right now
							end						
							offsetDirection = (blinkTarget.pos - t.pos):Normalized()
						--We will land behind our target relative to our starting position
						elseif blinkRange == -1 then						
							offsetDirection = (t.pos-blinkTarget.pos):Normalized()
						--They can choose which side of target to come out on , there is no way currently to read this data so we will only use this calculation if the spell radius is large
						elseif blinkRange == -255 then
							if radius > 250 then
								endPos = blinkTarget.pos
							end							
						end
						
						if offsetDirection then
							endPos = blinkTarget.pos - offsetDirection * blinkTarget.boundingRadius
						end
						
					end
				end	
				
				local interceptTime = self:GetSpellInterceptTime(myHero.pos, endPos, delay,speed)
				local deltaInterceptTime = interceptTime - windupRemaining
				if self:IsInRange(source, endPos, range) and deltaInterceptTime < timingAccuracy and (not checkCollision or not self:CheckMinionCollision(source, endPos, delay, speed, radius)) then
					target = t
					aimPosition = endPos
					return target,aimPosition					
				end
			end
		end
	end
end

function HPred:GetBlinkTarget(source, range, speed, delay, checkCollision, radius)
	local target
	local aimPosition
	for i = 1, LocalGameParticleCount() do 
		local particle = LocalGameParticle(i)
		if particle and _blinkLookupTable[particle.name] and self:IsInRange(source, particle.pos, range) then
			local pPos = particle.pos
			for k,v in pairs(self:GetEnemyHeroes()) do
				local t = v
				if t and t.isEnemy and self:IsInRange(t.pos, pPos, t.boundingRadius) then
					if (not checkCollision or not self:CheckMinionCollision(source, pPos, delay, speed, radius)) then
						target = t
						aimPosition = pPos
						return target,aimPosition
					end
				end
			end
		end
	end
end

function HPred:GetChannelingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	local target
	local aimPosition
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t then
			local interceptTime = self:GetSpellInterceptTime(myHero.pos, t.pos, delay, speed)
			if self:CanTarget(t) and self:IsInRange(source, t.pos, range) and self:IsChannelling(t, interceptTime) and (not checkCollision or not self:CheckMinionCollision(source, t.pos, delay, speed, radius)) then
				target = t
				aimPosition = t.pos	
				return target, aimPosition
			end
		end
	end
end

function HPred:GetImmobileTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)
	local target
	local aimPosition
	for i = 1, LocalGameHeroCount() do
		local t = LocalGameHero(i)
		if t and self:CanTarget(t) and self:IsInRange(source, t.pos, range) then
			local immobileTime = self:GetImmobileTime(t)
			
			local interceptTime = self:GetSpellInterceptTime(source, t.pos, delay, speed)
			if immobileTime - interceptTime > timingAccuracy and (not checkCollision or not self:CheckMinionCollision(source, t.pos, delay, speed, radius)) then
				target = t
				aimPosition = t.pos
				return target, aimPosition
			end
		end
	end
end

function HPred:CacheTeleports()
	--Get enemies who are teleporting to towers
--[[	for i = 1, LocalGameTurretCount() do
		local turret = LocalGameTurret(i);
		if turret and turret.isEnemy and not _cachedTeleports[turret.networkID] then
			local hasBuff, expiresAt = self:HasBuff(turret, "teleport_target")
			if hasBuff then
				self:RecordTeleport(turret, self:GetTeleportOffset(turret.pos,223.31),expiresAt)
			end
		end
	end	]]
	
	--Get enemies who are teleporting to wards	
--[[	for i = 1, LocalGameWardCount() do
		local ward = LocalGameWard(i);
		if ward and ward.isEnemy and not _cachedTeleports[ward.networkID] then
			local hasBuff, expiresAt = self:HasBuff(ward, "teleport_target")
			if hasBuff then
				self:RecordTeleport(ward, self:GetTeleportOffset(ward.pos,100.01),expiresAt)
			end
		end
	end]] -- Temporary disabled (crashes?)
	
	--Get enemies who are teleporting to minions
	for i = 1, LocalGameMinionCount() do
		local minion = LocalGameMinion(i);
		if minion and minion.isEnemy and not _cachedTeleports[minion.networkID] then
			local hasBuff, expiresAt = self:HasBuff(minion, "teleport_target")
			if hasBuff then
				self:RecordTeleport(minion, self:GetTeleportOffset(minion.pos,143.25),expiresAt)
			end
		end
	end
end

function HPred:RecordTeleport(target, aimPos, endTime)
	_cachedTeleports[target.networkID] = {}
	_cachedTeleports[target.networkID]["target"] = target
	_cachedTeleports[target.networkID]["aimPos"] = aimPos
	_cachedTeleports[target.networkID]["expireTime"] = endTime + LocalGameTimer()
end


function HPred:CalculateIncomingDamage()
	_incomingDamage = {}
	local currentTime = LocalGameTimer()
	for _, missile in pairs(_cachedMissiles) do
		if missile then 
			local dist = self:GetDistance(missile.data.pos, missile.target.pos)			
			if missile.name == "" or currentTime >= missile.timeout or dist < missile.target.boundingRadius then
				_cachedMissiles[_] = nil
			else
				if not _incomingDamage[missile.target.networkID] then
					_incomingDamage[missile.target.networkID] = missile.damage
				else
					_incomingDamage[missile.target.networkID] = _incomingDamage[missile.target.networkID] + missile.damage
				end
			end
		end
	end	
end

function HPred:GetIncomingDamage(target)
	local damage = 0
	if target and _incomingDamage[target.networkID] then
		damage = _incomingDamage[target.networkID]
	end
	return damage
end


local _maxCacheRange = 3000

--Right now only used to cache enemy windwalls
function HPred:CacheParticles()	
	if _windwall and _windwall.name == "" then
		_windwall = nil
	end
	
	for i = 1, LocalGameParticleCount() do
		local particle = LocalGameParticle(i)		
		if particle and self:IsInRange(particle.pos, myHero.pos, _maxCacheRange) then			
			if _find(particle.name, "W_windwall%d") and not _windwall then
				--We don't care about ally windwalls for now
				local owner =  self:GetObjectByHandle(particle.handle)
				if owner and owner.isEnemy then
					_windwall = particle
					_windwallStartPos = Vector(particle.pos.x, particle.pos.y, particle.pos.z)				
					
					local index = _len(particle.name) - 5
					local spellLevel = _sub(particle.name, index, index) -1 
					_windwallWidth = 150 + spellLevel * 25					
				end
			end
		end
	end
end

function HPred:CacheMissiles()
	local currentTime = LocalGameTimer()
	for i = 1, LocalGameMissileCount() do
		local missile = LocalGameMissile(i)
		if missile and not _cachedMissiles[missile.networkID] and missile.missileData then
			--Handle targeted missiles
			if missile.missileData.target and missile.missileData.owner then
				local missileName = missile.missileData.name
				local owner =  self:GetObjectByHandle(missile.missileData.owner)	
				local target =  self:GetObjectByHandle(missile.missileData.target)		
				if owner and target and _find(target.type, "Hero") then			
					--The missile is an auto attack of some sort that is targeting a player	
					if (_find(missileName, "BasicAttack") or _find(missileName, "CritAttack")) then
						--Cache it all and update the count
						_cachedMissiles[missile.networkID] = {}
						_cachedMissiles[missile.networkID].target = target
						_cachedMissiles[missile.networkID].data = missile
						_cachedMissiles[missile.networkID].danger = 1
						_cachedMissiles[missile.networkID].timeout = currentTime + 1.5
						
						local damage = owner.totalDamage
						if _find(missileName, "CritAttack") then
							--Leave it rough we're not that concerned
							damage = damage * 1.5
						end						
						_cachedMissiles[missile.networkID].damage = self:CalculatePhysicalDamage(target, damage)
					end
				end
			end
		end
	end
end

function HPred:CalculatePhysicalDamage(target, damage)
	
	local localDmg = 0
	if target and damage then		
		local targetArmor = target.armor * myHero.armorPenPercent - myHero.armorPen
		local damageReduction = 100 / ( 100 + targetArmor)
		if targetArmor < 0 then
			damageReduction = 2 - (100 / (100 - targetArmor))
		end		
		localDmg = damage * damageReduction
	end
	return localDmg
end

function HPred:CalculateMagicalDamage(target, damage)
	
	local localDmg = 0
	if target and damage then	
		local targetMR = target.magicResist * myHero.magicPenPercent - myHero.magicPen
		local damageReduction = 100 / ( 100 + targetMR)
		if targetMR < 0 then
			damageReduction = 2 - (100 / (100 - targetMR))
		end		
		localDmg = damage * damageReduction
	end
	
	return localDmg
end


function HPred:GetTeleportingTarget(source, range, delay, speed, timingAccuracy, checkCollision, radius)

	local target
	local aimPosition
	for _, teleport in pairs(_cachedTeleports) do
		if teleport and teleport.expireTime > LocalGameTimer() and self:IsInRange(source,teleport.aimPos, range) then			
			local spellInterceptTime = self:GetSpellInterceptTime(source, teleport.aimPos, delay, speed)
			local teleportRemaining = teleport.expireTime - LocalGameTimer()
			if spellInterceptTime > teleportRemaining and spellInterceptTime - teleportRemaining <= timingAccuracy and (not checkCollision or not self:CheckMinionCollision(source, teleport.aimPos, delay, speed, radius)) then								
				target = teleport.target
				aimPosition = teleport.aimPos
				return target, aimPosition
			end
		end
	end		
end

function HPred:GetTargetMS(target)
	if target then
		local ms = target.pathing.isDashing and target.pathing.dashSpeed or target.ms
		return ms
	end
	return _huge
end

function HPred:Angle(A, B)

	if A and B then
		local deltaPos = A - B
		local angle = _atan(deltaPos.x, deltaPos.z) *  180 / _pi	
		if angle < 0 then angle = angle + 360 end
		return angle
	end
end

function HPred:UpdateMovementHistory(unit)

	if not unit then return end

	if not _movementHistory[unit.charName] then
		_movementHistory[unit.charName] = {}
		_movementHistory[unit.charName]["EndPos"] = unit.pathing.endPos
		_movementHistory[unit.charName]["StartPos"] = unit.pathing.endPos
		_movementHistory[unit.charName]["PreviousAngle"] = 0
		_movementHistory[unit.charName]["ChangedAt"] = LocalGameTimer()
	end
		
	if _movementHistory[unit.charName]["EndPos"].x ~=unit.pathing.endPos.x or _movementHistory[unit.charName]["EndPos"].y ~=unit.pathing.endPos.y or _movementHistory[unit.charName]["EndPos"].z ~=unit.pathing.endPos.z then				
		_movementHistory[unit.charName]["PreviousAngle"] = self:Angle(Vector(_movementHistory[unit.charName]["StartPos"].x, _movementHistory[unit.charName]["StartPos"].y, _movementHistory[unit.charName]["StartPos"].z), Vector(_movementHistory[unit.charName]["EndPos"].x, _movementHistory[unit.charName]["EndPos"].y, _movementHistory[unit.charName]["EndPos"].z))
		_movementHistory[unit.charName]["EndPos"] = unit.pathing.endPos
		_movementHistory[unit.charName]["StartPos"] = unit.pos
		_movementHistory[unit.charName]["ChangedAt"] = LocalGameTimer()
	end
end

--Returns where the unit will be when the delay has passed given current pathing information. This assumes the target makes NO CHANGES during the delay.
function HPred:PredictUnitPosition(unit, delay)

	if not unit or not delay then return end
	local predictedPosition = unit.pos
	local timeRemaining = delay
	local pathNodes = self:GetPathNodes(unit)
	for i = 1, #pathNodes -1 do
		local nodeDistance = self:GetDistance(pathNodes[i], pathNodes[i +1])
		local nodeTraversalTime = nodeDistance / self:GetTargetMS(unit)
			
		if timeRemaining > nodeTraversalTime then
			--This node of the path will be completed before the delay has finished. Move on to the next node if one remains
			timeRemaining =  timeRemaining - nodeTraversalTime
			predictedPosition = pathNodes[i + 1]
		else
			local directionVector = (pathNodes[i+1] - pathNodes[i]):Normalized()
			predictedPosition = pathNodes[i] + directionVector *  self:GetTargetMS(unit) * timeRemaining
			break;
		end
	end
	return predictedPosition
end

function HPred:IsChannelling(target, interceptTime)
	if not target then return false end
	if target.activeSpell and target.activeSpell.valid and target.activeSpell.isChanneling then
		return true
	end
end

function HPred:HasBuff(target, buffName, minimumDuration)
	local duration = minimumDuration
	if not minimumDuration then
		duration = 0
	end
	if not target then return false end
	local durationRemaining
	for i = 1, target.buffCount do
		local buff = target:GetBuff(i)
		if buff and buff.duration > duration and buff.name == buffName then
			durationRemaining = buff.duration
			return true, durationRemaining
		end
	end
end

--Moves an origin towards the enemy team nexus by magnitude
function HPred:GetTeleportOffset(origin, magnitude)
	local teleportOffset = origin + (self:GetEnemyNexusPosition()- origin):Normalized() * magnitude
	return teleportOffset
end

function HPred:GetSpellInterceptTime(startPos, endPos, delay, speed)	
	local interceptTime = LocalGameLatency()/2000 + delay + self:GetDistance(startPos, endPos) / speed
	return interceptTime
end

--Checks if a target can be targeted by abilities or auto attacks currently.
--CanTarget(target)
	--target : gameObject we are trying to hit
function HPred:CanTarget(target)
	return target and target.isEnemy and target.alive and target.visible and target.isTargetable
end

--Derp: dont want to fuck with the isEnemy checks elsewhere. This will just let us know if the target can actually be hit by something even if its an ally
function HPred:CanTargetALL(target)
	return target and target.alive and target.visible and target.isTargetable
end

--Returns a position and radius in which the target could potentially move before the delay ends. ReactionTime defines how quick we expect the target to be able to change their current path
function HPred:UnitMovementBounds(unit, delay, reactionTime)

	if not unit then return end
	local startPosition = self:PredictUnitPosition(unit, delay)
	
	local radius = 0
	local deltaDelay = delay -reactionTime- self:GetImmobileTime(unit)	
	if (deltaDelay >0) then
		radius = self:GetTargetMS(unit) * deltaDelay	
	end
	return startPosition, radius	
end

--Returns how long (in seconds) the target will be unable to move from their current location
function HPred:GetImmobileTime(unit)

	if not unit then return 0 end
	local duration = 0
	for i = 0, unit.buffCount do
		local buff = unit:GetBuff(i);
		if buff and buff.count > 0 and buff.duration > duration and (buff.type == 5 or buff.type == 8 or buff.type == 21 or buff.type == 22 or buff.type == 24 or buff.type == 11 or buff.type == 29 or buff.type == 30 or buff.type == 39 ) then
			duration = buff.duration
		end
	end
	return duration		
end

--Returns how long (in seconds) the target will be slowed for
function HPred:GetSlowedTime(unit)

	if not unit then return 0 end
	local duration = 0
	for i = 0, unit.buffCount do
		local buff = unit:GetBuff(i);
		if buff and buff.count > 0 and buff.duration > duration and buff.type == 10 then
			duration = buff.duration			
			return duration
		end
	end
	return duration		
end

--Returns all existing path nodes
function HPred:GetPathNodes(unit)
	local nodes = {}
	_insert(nodes, unit.pos)
	if unit and unit.pathing and unit.pathing.hasMovePath then
		for i = unit.pathing.pathIndex, unit.pathing.pathCount do
			path = unit:GetPath(i)
			_insert(nodes, path)
		end
	end		
	return nodes
end

--Finds any game object with the correct handle to match (hero, minion, wards on either team)
function HPred:GetObjectByHandle(handle)

	if not handle then return nil end

	local target
	for i = 1, LocalGameHeroCount() do
		local enemy = LocalGameHero(i)
		if enemy and enemy.handle == handle then
			target = enemy
			return target
		end
	end
	
	for i = 1, LocalGameMinionCount() do
		local minion = LocalGameMinion(i)
		if minion and minion.handle == handle then
			target = minion
			return target
		end
	end
	
--[[	for i = 1, LocalGameWardCount() do
		local ward = LocalGameWard(i);
		if ward and ward.handle == handle then
			target = ward
			return target
		end
	end]]
	
--[[	for i = 1, LocalGameTurretCount() do 
		local turret = LocalGameTurret(i)
		if turret and turret.handle == handle then
			target = turret
			return target
		end
	end]]
	
	for i = 1, Game.ParticleCount() do 
		local particle = Game.Particle(i)
		if particle and particle.handle == handle then
			target = particle
			return target
		end
	end
end

function HPred:GetHeroByPosition(position)

	if not position then return nil end
	local target
	for i = 1, LocalGameHeroCount() do
		local enemy = LocalGameHero(i)
		if enemy and enemy.pos.x == position.x and enemy.pos.y == position.y and enemy.pos.z == position.z then
			target = enemy
			return target
		end
	end
end

function HPred:GetObjectByPosition(position)

	if not position then return nil end
	local target
	for i = 1, LocalGameHeroCount() do
		local enemy = LocalGameHero(i)
		if enemy and enemy.pos.x == position.x and enemy.pos.y == position.y and enemy.pos.z == position.z then
			target = enemy
			return target
		end
	end
	
	for i = 1, LocalGameMinionCount() do
		local enemy = LocalGameMinion(i)
		if enemy and enemy.pos.x == position.x and enemy.pos.y == position.y and enemy.pos.z == position.z then
			target = enemy
			return target
		end
	end
--[[	
	for i = 1, LocalGameWardCount() do
		local enemy = LocalGameWard()
		if enemy and enemy.pos.x == position.x and enemy.pos.y == position.y and enemy.pos.z == position.z then
			target = enemy
			return target
		end
	end]]
	
	for i = 1, LocalGameParticleCount() do 
		local enemy = LocalGameParticle()
		if enemy and enemy.pos.x == position.x and enemy.pos.y == position.y and enemy.pos.z == position.z then
			target = enemy
			return target
		end
	end
end

function HPred:GetEnemyHeroByHandle(handle)	

	if not handle then return nil end
	local target
	for i = 1, LocalGameHeroCount() do
		local enemy = LocalGameHero(i)
		if enemy and enemy.handle == handle then
			target = enemy
			return target
		end
	end
end

--Finds the closest particle to the origin that is contained in the names array
function HPred:GetNearestParticleByNames(origin, names)
	local target
	local distance = 999999
	for i = 1, LocalGameParticleCount() do 
		local particle = LocalGameParticle(i)
		if particle then 
			local d = self:GetDistance(origin, particle.pos)
			if d < distance then
				distance = d
				target = particle
			end
		end
	end
	return target, distance
end

--Returns the total distance of our current path so we can calculate how long it will take to complete
function HPred:GetPathLength(nodes)
	if not nodes then return 0 end
	local result = 0
	for i = 1, #nodes -1 do
		result = result + self:GetDistance(nodes[i], nodes[i + 1])
	end
	return result
end


--I know this isn't efficient but it works accurately... Leaving it for now.
function HPred:CheckMinionCollision(origin, endPos, delay, speed, radius, frequency)
	
	if origin and endpos and radius then
		if not frequency then
			frequency = radius
		end
		local directionVector = (endPos - origin):Normalized()
		local checkCount = self:GetDistance(origin, endPos) / frequency
		for i = 1, checkCount do
			local checkPosition = origin + directionVector * i * frequency
			local checkDelay = delay + self:GetDistance(origin, checkPosition) / speed
			if self:IsMinionIntersection(checkPosition, radius, checkDelay, radius * 3) then
				return true
			end
		end
	end
	return false
end


function HPred:IsMinionIntersection(location, radius, delay, maxDistance)

	if location and radius and delay then
		if not maxDistance then
			maxDistance = 500
		end
		for i = 1, Game.MinionCount() do
			local minion = Game.Minion(i)
			if minion and self:CanTarget(minion) and self:IsInRange(minion.pos, location, maxDistance) then
				local predictedPosition = self:PredictUnitPosition(minion, delay)
				if self:IsInRange(location, predictedPosition, radius + minion.boundingRadius) then
					return true
				end
			end
		end
	end
	return false
end

function HPred:VectorPointProjectionOnLineSegment(v1, v2, v)
	if v1 and v2 and v then
		assert(v1 and v2 and v, "VectorPointProjectionOnLineSegment: wrong argument types (3 <Vector> expected)")
		local cx, cy, ax, ay, bx, by = v.x, (v.z or v.y), v1.x, (v1.z or v1.y), v2.x, (v2.z or v2.y)
		local rL = ((cx - ax) * (bx - ax) + (cy - ay) * (by - ay)) / ((bx - ax) * (bx - ax) + (by - ay) * (by - ay))
		local pointLine = { x = ax + rL * (bx - ax), y = ay + rL * (by - ay) }
		local rS = rL < 0 and 0 or (rL > 1 and 1 or rL)
		local isOnSegment = rS == rL
		local pointSegment = isOnSegment and pointLine or { x = ax + rS * (bx - ax), y = ay + rS * (by - ay) }
	end
	return pointSegment, pointLine, isOnSegment
end

--Determines if there is a windwall between the source and target pos. 
function HPred:IsWindwallBlocking(source, target)
	if _windwall and source and target then
		local windwallFacing = (_windwallStartPos-_windwall.pos):Normalized()
		return self:DoLineSegmentsIntersect(source, target, _windwall.pos + windwallFacing:Perpendicular() * _windwallWidth, _windwall.pos + windwallFacing:Perpendicular2() * _windwallWidth)
	end	
	return false
end
--Returns if two line segments cross eachother. AB is segment 1, CD is segment 2.
function HPred:DoLineSegmentsIntersect(A, B, C, D)

	local o1 = self:GetOrientation(A, B, C)
	local o2 = self:GetOrientation(A, B, D)
	local o3 = self:GetOrientation(C, D, A)
	local o4 = self:GetOrientation(C, D, B)
	
	if o1 ~= o2 and o3 ~= o4 then
		return true
	end
	
	if o1 == 0 and self:IsOnSegment(A, C, B) then return true end
	if o2 == 0 and self:IsOnSegment(A, D, B) then return true end
	if o3 == 0 and self:IsOnSegment(C, A, D) then return true end
	if o4 == 0 and self:IsOnSegment(C, B, D) then return true end
	
	return false
end

--Determines the orientation of ordered triplet
--0 = Colinear
--1 = Clockwise
--2 = CounterClockwise
function HPred:GetOrientation(A,B,C)
	if A and B and C then
		local val = (B.z - A.z) * (C.x - B.x) -
			(B.x - A.x) * (C.z - B.z)
		if val == 0 then
			return 0
		elseif val > 0 then
			return 1
		else
			return 2
		end
	end
	return 0
end

function HPred:IsOnSegment(A, B, C)
	return B.x <= _max(A.x, C.x) and 
		B.x >= _min(A.x, C.x) and
		B.z <= _max(A.z, C.z) and
		B.z >= _min(A.z, C.z)
end

--Gets the slope between two vectors. Ignores Y because it is non-needed height data. Its all 2d math.
function HPred:GetSlope(A, B)
	return (B.z - A.z) / (B.x - A.x)
end

function HPred:GetEnemyByName(name)
	local target
	if name then
		for i = 1, LocalGameHeroCount() do
			local enemy = LocalGameHero(i)
			if enemy and enemy.isEnemy and enemy.charName == name then
				target = enemy
				return target
			end
		end
	end
	return nil
end

function HPred:IsPointInArc(source, origin, target, angle, range)
	if origin and target and source and angle and range then
		local deltaAngle = _abs(HPred:Angle(origin, target) - HPred:Angle(source, origin))
		if deltaAngle < angle and self:IsInRange(origin,target,range) then
			return true
		end
	end
	return false
end

function HPred:GetEnemyHeroes()
	return Utils:GetEnemyHeroes()
end -- #Lazy

function HPred:GetDistanceSqr(p1, p2)	
	return Utils:GetDistanceSqr(p1, p2)
end -- #Lazy

function HPred:IsInRange(p1, p2, range)
	return self:GetDistance(p1, p2) <= range
end

function HPred:GetDistance(p1, p2)
	return _sqrt(self:GetDistanceSqr(p1, p2))
end

--[[print("Q Name: " ..myHero:GetSpellData(_Q).name..  " | Q Range: " ..myHero:GetSpellData(_Q).range.. " | Q Width: " ..myHero:GetSpellData(_Q).width.. " | Q Speed: " ..myHero:GetSpellData(_Q).speed)
print("W Name: " ..myHero:GetSpellData(_W).name..  " | W Range: " ..myHero:GetSpellData(_W).range.. " | W Width: " ..myHero:GetSpellData(_W).width.. " | W Speed: " ..myHero:GetSpellData(_W).speed)
print("E Name: " ..myHero:GetSpellData(_E).name..  " | E Range: " ..myHero:GetSpellData(_E).range.. " | E Width: " ..myHero:GetSpellData(_E).width.. " | E Speed: " ..myHero:GetSpellData(_E).speed)
print("R Name: " ..myHero:GetSpellData(_R).name..  " | R Range: " ..myHero:GetSpellData(_R).range.. " | R Width: " ..myHero:GetSpellData(_R).width.. " | R Speed: " ..myHero:GetSpellData(_R).speed)]]

-- Last but not least https://i.imgur.com/rQJRA8u.png 
