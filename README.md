# SQL_StoredProcedures


# ---This Procedure gives Batsman score 

Select AllPlayers.FTID 

,AllPlayers.PDesc

,sum(Runs) Rn,  
--round(AVG(SR),2) SR, 
case when sum(BF) = 0 then 0  else round(cast(sum(Runs)*100 as float)/sum(BF),2) end as SR,
--sum(BF) BF,

sum(Bds) Bds, sum([40s]) [40s],

sum(Dis) Ds,

sum(Bs) BB, 
-- sum(DB) DB,
case when sum(Bs) = 0 then 0 else round((cast(sum(RG) as float)/(sum(Bs)/6)),2) end as EC, 
sum(Wt) WT,
sum([3ws]) [3WS]
--sum(RG) RG,


 from 
(
select distinct FTID,PID,PDesc from SSISFP where Starter = 1 
)AllPlayers 

--Batting

Left Join
(Select F.Description, sum(B.Runs) Runs, round(AVG(strikerate),2) SR ,(sum(Sixes)*2) + sum(Fours) [Bds], sum(BallsFaced) bf, 
sum(case when Runs >= 40 then 1 else 0 end)
+ sum(case when Runs >= 100 then 1 else 0 end) as [40s]
, P.PDesc, F.FTID,P.PID
from SSISFP P 
join SSISFT F on F.FTID = P.FTID
join SSISBT B on P.PID = B.PlayerID 
join SSISMT M on M.UniqueID = B.UniqueID 
where P.Starter = 1 and cast(M.PlayDate as date) between cast(P.ActiveDate as Date) and cast(P.ExpirationDate as date)
Group by F.FTID, F.Description, P.PDesc,P.PID) Batting on AllPlayers.PID = Batting.pid and AllPlayers.FTID = Batting.FTID

--Bowling

left join (Select F.Description, 
sum(dbo.fnGetBs(Bw.OversBowled)) Bs, sum(Bw.DotBalls) as Dts, sum(RunsGiven) RG
, case when sum(dbo.fnGetBs(Bw.OversBowled)) = 0 then 0 else round(cast(sum(RunsGiven) as float)/(sum(dbo.fnGetBs(Bw.OversBowled))/6), 2) end as Ec
, sum(DotBalls) DB
,sum(WicketsTaken) Wt, 
sum(case when WicketsTaken >= 3 then 1 else 0 end)
+ sum(case when WicketsTaken >= 5 then 1 else 0 end) as [3Ws]
, P.PDesc, F.FTID, P.PID
from SSISFP P 
join SSISFT F on F.FTID = P.FTID
join SSISBW Bw on P.PID = Bw.PlayerID 
join SSISMT M on M.UniqueID = Bw.UniqueID 
where P.Starter = 1 and cast(M.PlayDate as date) between cast(P.ActiveDate as Date) and cast(P.ExpirationDate as date)
Group by F.FTID, F.Description, P.PDesc,p.PID) Bowling on Bowling.PID = AllPlayers.PID and AllPlayers.FTID = Bowling.FTID

--Fielding

left join (Select F.Description, 

sum(FD.RunOut + FD.Stumped + FD.Catches) Dis

, P.PDesc, F.FTID, P.PID
from SSISFP P 
join SSISFT F on F.FTID = P.FTID
join SSISFD FD on P.PID = FD.PlayerID
join SSISMT M on M.UniqueID = FD.UniqueID 
where P.Starter = 1 and cast(M.PlayDate as date) between cast(P.ActiveDate as Date) and cast(P.ExpirationDate as date)
Group by F.FTID, F.Description, P.PDesc,P.PID) Fielding on Fielding.PID = AllPlayers.PID and AllPlayers.FTID = Fielding.FTID

Group by AllPlayers.FTID

, AllPlayers.PDesc


--) Scrs
--) OScrs
Order by FTID



# ---This procedure gives ranks for tournament

Select *, 
round(cast((RNR+SRR+BDR+FTR+DSR+BBR+ECR+WTR+[3WR]) as float)/9,2) OAVG,
Rank() Over(Order By round(cast((RNR+SRR+BDR+FTR+DSR+BBR+ECR+WTR+[3WR]) as float)/9,2) desc) as OVR 

from 
(
select *, 
RANK() OVER(ORDER BY Rn) AS RNR, 
RANK() OVER(ORDER BY SR) AS SRR,
RANK() OVER(ORDER BY Bds) AS BDR,
RANK() OVER(ORDER BY [40s]) AS FTR,
RANK() OVER(ORDER BY Ds) AS DSR,
RANK() OVER(ORDER BY BB) AS BBR,
RANK() OVER(ORDER BY EC desc) AS ECR,
RANK() OVER(ORDER BY WT) AS WTR,
RANK() OVER(ORDER BY [3WS]) AS [3WR]

from 


(
-- Run From here for individual totals

Select AllPlayers.FTID 

--,AllPlayers.PDesc

,sum(Runs) Rn,  
--round(AVG(SR),2) SR, 
case when sum(BF) = 0 then 0  else round(cast(sum(Runs)*100 as float)/sum(BF),2) end as SR,
--sum(BF) BF,

sum(Bds) Bds, sum([40s]) [40s],

sum(Dis) Ds,

sum(Bs) BB, 
-- sum(DB) DB,
case when sum(Bs) = 0 then 0 else round((cast(sum(RG) as float)/(sum(Bs)/6)),2) end as EC, 
sum(Wt) WT,
sum([3ws]) [3WS]
--sum(RG) RG,


 from 
(
select distinct FTID,PID,PDesc from SSISFP where Starter = 1 
)AllPlayers 

--Batting
Left Join
(Select F.Description, sum(B.Runs) Runs, round(AVG(strikerate),2) SR ,(sum(Sixes)*2) + sum(Fours) [Bds], sum(BallsFaced) bf, 
sum(case when Runs >= 40 then 1 else 0 end)
+ sum(case when Runs >= 100 then 1 else 0 end) as [40s]
, P.PDesc, F.FTID,P.PID
from SSISFP P 
join SSISFT F on F.FTID = P.FTID
join SSISBT B on P.PID = B.PlayerID 
join SSISMT M on M.UniqueID = B.UniqueID 
where P.Starter = 1 and cast(M.PlayDate as date) between cast(P.ActiveDate as Date) and cast(P.ExpirationDate as date)
Group by F.FTID, F.Description, P.PDesc,P.PID) Batting on AllPlayers.PID = Batting.pid and AllPlayers.FTID = Batting.FTID

--Bowling

left join (Select F.Description, 
sum(dbo.fnGetBs(Bw.OversBowled)) Bs, sum(Bw.DotBalls) as Dts, sum(RunsGiven) RG
, case when sum(dbo.fnGetBs(Bw.OversBowled)) = 0 then 0 else round(cast(sum(RunsGiven) as float)/(sum(dbo.fnGetBs(Bw.OversBowled))/6), 2) end as Ec
, sum(DotBalls) DB
,sum(WicketsTaken) Wt, 
sum(case when WicketsTaken >= 3 then 1 else 0 end)
+ sum(case when WicketsTaken >= 5 then 1 else 0 end) as [3Ws]
, P.PDesc, F.FTID, P.PID
from SSISFP P 
join SSISFT F on F.FTID = P.FTID
join SSISBW Bw on P.PID = Bw.PlayerID 
join SSISMT M on M.UniqueID = Bw.UniqueID 
where P.Starter = 1 and cast(M.PlayDate as date) between cast(P.ActiveDate as Date) and cast(P.ExpirationDate as date)
Group by F.FTID, F.Description, P.PDesc,p.PID) Bowling on Bowling.PID = AllPlayers.PID and AllPlayers.FTID = Bowling.FTID

--Fielding

left join (Select F.Description, 

sum(FD.RunOut + FD.Stumped + FD.Catches) Dis

, P.PDesc, F.FTID, P.PID
from SSISFP P 
join SSISFT F on F.FTID = P.FTID
join SSISFD FD on P.PID = FD.PlayerID
join SSISMT M on M.UniqueID = FD.UniqueID 
where P.Starter = 1 and cast(M.PlayDate as date) between cast(P.ActiveDate as Date) and cast(P.ExpirationDate as date)
Group by F.FTID, F.Description, P.PDesc,P.PID) Fielding on Fielding.PID = AllPlayers.PID and AllPlayers.FTID = Fielding.FTID

Group by AllPlayers.FTID

--, AllPlayers.PDesc


) Scrs
) OScrs
Order by FTID

