```

CREATE OR REPLACE FUNCTION public.check_children_done(node_id integer)
 RETURNS boolean
 LANGUAGE plpgsql
AS $function$

                        declare

                            begin

                                

                            if exists (
select
	id
from
	csf_manager_node
where
	parent_id = node_id
	and done = false
	and id != node_id
limit 1) then

return false ;
else

return true ;
end if;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.check_children_done_purpose(_purpose_id integer)
 RETURNS boolean
 LANGUAGE plpgsql
AS $function$

declare

	list_nodes_bro int[];

node_id_ int4;

begin


select
	node_id
into
	node_id_
from
	csf_manager_purposenode
where
	id = _purpose_id;

raise notice 'node_id  %',
node_id_;

list_nodes_bro := array(
select
	id
from
	csf_manager_node
where
	parent_id = node_id_ ) ;

raise notice 'list_nodes_bro  %',
list_nodes_bro;

if exists (
select
	id
from
	csf_manager_purposenode
where
	node_id = any(list_nodes_bro)
		and done = false
	limit 1 

    ) then

return false ;
else

return true ;
end if;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.check_children_done_purpose2(_purpose_id integer)
 RETURNS boolean
 LANGUAGE plpgsql
AS $function$

declare
node_id int ;
    begin

select node_id into node_id from csf_manager_purposenode where id = _purpose_id ; 

    if exists (
select
	id
from
	csf_manager_node
where
	parent_id = node_id
	and done = false
	and id != node_id
limit 1) then

return false ;
else

return true ;
end if;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.check_have_bro_incomplete_or_id_bro(node_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$

                        declare
                            id_bro int;

begin

                            
                        select
	id
into
	id_bro
from
	csf_manager_node
where
	parent_id = (
	select
		parent_id
	from
		csf_manager_node
	where
		id = node_id)
	and done = false
	and id != node_id
limit 1 ;

if not found then


return 0 ;
else

return id_bro ;
end if;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.compute_child(value_parent_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$

declare

childs record ;

rec record;

parent_is_positive bool = false ;

total numeric(7,4) = 0;

sum smallint = 0;

relation_sum numeric(7,4) = 0;

begin

select positive into parent_is_positive from csf_manager_node  where id=value_parent_id ;
		
raise notice 'parent_is_positive % ----- value_parent_id %',parent_is_positive,value_parent_id;
drop table if exists temp_relation;

create temporary table temp_relation(
    
 dest_id serial4 not null,
 score int2 not null ,
 positive bool NOT NULL
 ) ;
insert
	into
	temp_relation (
	dest_id,
	score,
	positive
	)

select
	source_node_id ,
	score,
	positive
from
	csf_manager_treesrelation
where
	destination_node_id = value_parent_id ;

select
	sum(score)
into
	sum
from
	csf_manager_node
where
	parent_id = value_parent_id and positive = parent_is_positive ;


if sum is null then

sum = 0 ;
end if ;


raise notice 'sum______ %   -  ' , sum ;

select
	sum(score)
into
	relation_sum
from
	temp_relation where positive = parent_is_positive
 ;

if relation_sum is null then

relation_sum = 0 ;
end if ;


sum = sum + relation_sum ;

raise notice 'sum %   -    -   %' , sum ,relation_sum ;

if sum = 0 then

return total ;
end if ;

for childs in


select
	temp_relation.score as scores ,
	temp_relation.dest_id as ids ,
	temp_relation.positive as positives ,
	csf_manager_node.probability as probabilitys
from
	temp_relation
join csf_manager_node on
	temp_relation.dest_id = csf_manager_node.id 

loop

if childs.positives <> parent_is_positive then 

total = -((childs.probabilitys * childs.scores) / sum )+ total ;

else 

total = ((childs.probabilitys * childs.scores) / sum )+ total ;

end if ; 



end loop;

for rec in
select
	*
from
	csf_manager_node
where
	parent_id = value_parent_id 

loop

--	total = ((rec.probability * rec.score) / sum )+ total ;


if rec.positive <> parent_is_positive then 

total = -((rec.probability * rec.score) / sum )+ total ;

else 

total = ((rec.probability * rec.score) / sum )+ total ;

end if ; 



end loop;

if total < 0 then
total = 0 ;
end if ;

raise notice 'total %',
total;
return total ;
end;

$function$
;


CREATE OR REPLACE FUNCTION public.compute_purpose_child(_value_parent_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$

declare

childs record ;

rec record;
purpose record ;
node record ;
total smallint = 0;

sum smallint = 0;

relation_sum smallint = 0;

begin

select * into purpose from csf_manager_purposenode   where id=_value_parent_id ;

select * into node from csf_manager_node where id=purpose.node_id ;
		
raise notice 'parent_is_positive % ----- _value_parent_id %',purpose.positive,purpose.id;
drop table if exists temp_relation;

create temporary table temp_relation(
    
 dest_id serial4 not null,
 score int2 not null ,
 positive bool NOT NULL
 ) ;
insert
	into
	temp_relation (
	dest_id,
	score,
	positive
	)

select
	source_node_id ,
	score,
	positive
from
	csf_manager_treesrelation
where
	destination_node_id = purpose.node_id ;



select
	sum(cmp.score)
into
	sum
from
	csf_manager_node cmn join csf_manager_purposenode cmp on cmn.id =cmp.node_id 
where
	cmn.parent_id = node.id and cmp.positive = purpose.positive ;






if sum is null then

sum = 0 ;
end if ;


raise notice 'sum______ %   -  ' , sum ;

select
	sum(score)
into
	relation_sum
from
	temp_relation where positive = purpose.positive
 ;

if relation_sum is null then

relation_sum = 0 ;
end if ;


sum = sum + relation_sum ;

raise notice 'sum %   -    -   %' , sum ,relation_sum ;

if sum = 0 then

return total ;
end if ;

for childs in


select
	temp_relation.score as scores ,
	temp_relation.dest_id as ids ,
	temp_relation.positive as positives ,
	csf_manager_purposenode.probability as probabilitys
from
	temp_relation
join csf_manager_purposenode on
	temp_relation.dest_id = csf_manager_purposenode.node_id  

loop
raise notice 'childs';
	
raise notice 'childs ____ %',childs;

if childs.positives <> purpose.positive then 

total = -((childs.probabilitys * childs.scores) / sum )+ total ;

else 

total = ((childs.probabilitys * childs.scores) / sum )+ total ;

end if ; 



end loop;

for rec in
select
	cmp.probability as probability_ ,cmp.score as score_ , cmp.positive as positive_
from
	csf_manager_node cmn join csf_manager_purposenode cmp on cmn.id =cmp.node_id 
where
	cmn.parent_id = node.id 

loop

--	total = ((rec.probability * rec.score) / sum )+ total ;
raise notice 'rec';
raise notice 'rec ____ %',rec;
if rec.positive_ <> purpose.positive then 

total = -((rec.probability_ * rec.score_) / sum )+ total ;

else 

total = ((rec.probability_ * rec.score_) / sum )+ total ;

end if ; 



end loop;

if total < 0 then
total = 0 ;
end if ;

raise notice 'total %',
total;
return total ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.compute_total_purpose_severity(_list_purpose integer[])
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare
rec record;

total smallint := 0;

purpose int;

begin
	foreach purpose in array _list_purpose

	  loop
for rec in 
select
	csf_manager_purposenode_event.event_id ,
	threats_event.id ,
	csf_manager_purposenode_event.purposenode_id ,
	threats_event.severity_according_expire_date as sev
from
	public.csf_manager_purposenode_event
inner join threats_event on
	csf_manager_purposenode_event.event_id = threats_event.id
where
	 csf_manager_purposenode_event.purposenode_id = purpose

loop

total = ((100 - total) * rec.sev / 100) + total ;
end loop;

if total is null then

total = 0 ;
end if ;
raise notice '%',total;
update
	csf_manager_purposenode 
set
	event_effect_percent = total
where
	id = purpose;

total = 0 ;
end loop;
end;

$function$
;


CREATE OR REPLACE FUNCTION public.compute_total_severity(list_node integer[])
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare
rec record;

total numeric(7,4) := 0;

node int;

begin
	foreach node in array list_node

	  loop
for rec in 
select
	csf_manager_node_event.event_id ,
	threats_event.id ,
	csf_manager_node_event.node_id ,
	threats_event.severity_according_expire_date as sev
from
	public.csf_manager_node_event
inner join threats_event on
	csf_manager_node_event.event_id = threats_event.id
where
	 csf_manager_node_event.node_id = node

loop

total = ((100 - total) * rec.sev / 100) + total ;
end loop;

if total is null then

total = 0 ;
end if ;

update
	csf_manager_node
set
	event_effect_percent = total
where
	id = node;

total = 0 ;
end loop;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.filter_event_except_category(titles character varying, actor character varying[] DEFAULT '{}'::character varying[], csf integer[] DEFAULT '{}'::integer[], pos integer DEFAULT 0, endtdate date DEFAULT NULL::date, startdate date DEFAULT NULL::date, keyword character varying[] DEFAULT '{}'::character varying[], target character varying[] DEFAULT '{}'::character varying[], importance integer[] DEFAULT '{}'::integer[], varisexpired boolean DEFAULT NULL::boolean)
 RETURNS TABLE(id integer, title character varying, summary text, category character varying, parentcategory character varying, image character varying, update_date timestamp with time zone)
 LANGUAGE plpgsql
AS $function$
	declare 
		actors integer[];
		csfs integer[];
		start_date date;
		end_date date;
		currentdate date;
		keywords varchar[];
		targets varchar[];
		importances integer[];
	begin
		currentdate = current_date;
		raise notice 'title: %', titles;
		raise notice 'ator: %', actor;
		raise notice 'csf: %', csf;
		raise notice 'pos: %', pos;
		raise notice 'start: %', startdate;
		raise notice 'end: %', endtdate;
		raise notice 'keywords: %', keyword;
		raise notice 'importance: %',importance;
		if startdate notnull then
			start_date = startdate;
		else
			start_date = currentdate - integer '7' ;
		end if;
	
		if endtdate notnull then
			end_date = endtdate;
		else 
			end_date =	currentdate ; 
		end if;
	
	
		if array_length(actor,1) >= 1 then 
			actors := array (select threats_actor.id from threats_actor where threats_actor.name = any(actor));
		else
			actors := array(select threats_actor.id from threats_actor);
		end if;
		
		if array_length(keyword,1) >= 1 then
			keywords := array (select threats_kyword.name from threats_kyword where threats_kyword.name = any(keyword));
		else
			keywords := array (select threats_kyword.name from threats_kyword);
		end if;
	
		if array_length(target,1) >= 1 then
			targets := array (select threats_target.name from threats_target where threats_target.name = any(target));
		else
			targets := array (select threats_target.name from threats_target);
		end if;
	
		if array_length(importance,1) >= 1 then 
			importances := importance;
		else
			importances := array[1,2,3,4];
		end if;
		
		if array_length(csf,1) >= 1 then 
			csfs := array (select threats_csf.id from threats_csf join csf_manager_node on threats_csf.id = csf_manager_node.csf_id 
		where
			csf_manager_node.id = any(csf));
		else 
			csfs := array(select threats_csf.id from threats_csf join csf_manager_node on threats_csf.id = csf_manager_node.csf_id)
		;end if;
	
		if pos <> 0 then
		raise notice 'ssssssssssssss: %',importance;
			return query
			select distinct threats_event.id ,threats_event.title,threats_event.summary , selfcat.name,parentcat.name, threats_event.image , threats_event.update_date from threats_event 
			
				join threats_event_event_scope esc on esc.event_id = threats_event.id
				join threats_event_event_scope est on est.event_id = threats_event.id
				
				join threats_scope sc on esc.scope_id = sc.id
				join threats_scope st on est.scope_id = st.id
				join threats_subcsfratevalue fc on fc.id = sc.subcsfratevalue_id
				join threats_subcsfratevalue ft on ft.id = st.subcsfratevalue_id
				
				left join threats_category selfcat on fc.category_id = selfcat.id
				
				
				left join threats_target on ft.target_id  = threats_target.id
			
				join threats_threatsevent2 on threats_event.id = threats_threatsevent2.event_id 
				join threats_tnapcy_actors on threats_threatsevent2.tnapcy_id = threats_tnapcy_actors.tnapcy_id
				join threats_tnapcy_keywords on threats_tnapcy_keywords.tnapcy_id = threats_threatsevent2.tnapcy_id
				join threats_kyword on threats_kyword.id = threats_tnapcy_keywords.kyword_id
				join threats_threatsevent2_postition on threats_threatsevent2_postition.threatsevent2_id = threats_threatsevent2.id
				join threats_actor on threats_tnapcy_actors.actor_id = threats_actor.id
				
				join csf_manager_node n1 on n1.csf_id = threats_event.csf_id
				inner join csf_manager_node n2 on n2.id = n1.parent_id
				 join threats_csf on n2.csf_id = threats_csf.id
				 join threats_subcsfrate on n2.csf_id = threats_subcsfrate.csf_id
				left join threats_category parentcat on threats_subcsfrate.category_id = parentcat.id
				
			where threats_event.title like concat('%',titles,'%') 
				and threats_actor.id = any(actors) 
				and threats_event.csf_id = any(csfs) 
				and threats_threatsevent2_postition.postition_id = pos 
				and threats_event.create_date::date >= start_date 
				and threats_event.create_date::date <= end_date 
				and threats_kyword.name = any(keywords)
				and threats_target.name = any(targets)
				and threats_event.is_expired = coalesce (varisexpired ,threats_event.is_expired)
				and threats_event.importance = any(importances) order by threats_event.update_date desc ;
		else
			return query
			select distinct threats_event.id ,threats_event.title,threats_event.summary , selfcat.name,parentcat.name, threats_event.image , threats_event.update_date from threats_event 
			
				join threats_event_event_scope esc on esc.event_id = threats_event.id
				join threats_event_event_scope est on est.event_id = threats_event.id
				
				join threats_scope sc on esc.scope_id = sc.id
				join threats_scope st on est.scope_id = st.id
				join threats_subcsfratevalue fc on fc.id = sc.subcsfratevalue_id
				join threats_subcsfratevalue ft on ft.id = st.subcsfratevalue_id
				
				left join threats_category selfcat on fc.category_id = selfcat.id
				
				
				left join threats_target on ft.target_id  = threats_target.id
			
				join threats_threatsevent2 on threats_event.id = threats_threatsevent2.event_id 
				join threats_tnapcy_actors on threats_threatsevent2.tnapcy_id = threats_tnapcy_actors.tnapcy_id 
				join threats_actor on threats_tnapcy_actors.actor_id = threats_actor.id
				join threats_tnapcy_keywords on threats_tnapcy_keywords.tnapcy_id = threats_threatsevent2.tnapcy_id
				join threats_kyword on threats_kyword.id = threats_tnapcy_keywords.kyword_id
				
				
				join csf_manager_node n1 on n1.csf_id = threats_event.csf_id
				inner join csf_manager_node n2 on n2.id = n1.parent_id
				 join threats_csf on n2.csf_id = threats_csf.id
				 join threats_subcsfrate on n2.csf_id = threats_subcsfrate.csf_id
				left join threats_category parentcat on threats_subcsfrate.category_id = parentcat.id
				
			where threats_event.title like concat('%',titles,'%') 
				and threats_actor.id = any(actors) and threats_event.csf_id = any(csfs) 
				and threats_event.create_date::date >= start_date 
				and threats_event.create_date::date <= end_date	
				and threats_kyword.name = any(keywords)
				and threats_target.name = any(targets)
				and threats_event.is_expired = coalesce (varisexpired ,threats_event.is_expired)
				and threats_event.importance = any(importances) order by threats_event.update_date desc  ;
		end if;
	END;
$function$
;

CREATE OR REPLACE FUNCTION public.filter_event_function(titles character varying, actor character varying[] DEFAULT '{}'::character varying[], csf integer[] DEFAULT '{}'::integer[], pos integer DEFAULT 0, endtdate date DEFAULT NULL::date, startdate date DEFAULT NULL::date, keyword character varying[] DEFAULT '{}'::character varying[], target character varying[] DEFAULT '{}'::character varying[], importance integer[] DEFAULT '{}'::integer[], categories character varying[] DEFAULT '{}'::character varying[], varisexpired boolean DEFAULT NULL::boolean)
 RETURNS TABLE(id integer, title character varying, summary text, category character varying, image character varying, update_date timestamp with time zone)
 LANGUAGE plpgsql
AS $function$
	declare rec record;
	event_ids integer[];
	category varchar;
	BEGIN
		create temporary table if not exists eventstemp(id integer ,title varchar,summary text,category varchar,image varchar , update_date timestamptz);
		delete from eventstemp;
		For rec in select * from public.filter_event_except_category(titles,actor,csf,pos,endtdate,startdate,keyword,target,importance,varisexpired) loop
			if exists (select * from eventstemp where eventstemp.id = rec.id) then 
				if rec.category is not null then 
					update eventstemp set category = rec.category where eventstemp.id = rec.id;
				else
					update eventstemp set category = rec.parentcategory where eventstemp.id = rec.id and eventstemp.category is null;
				end if;
			else
				if rec.category is not null then 
					insert into eventstemp(id, title , summary , category , image , update_date) values (rec.id, rec.title,rec.summary,rec.category , rec.image , rec.update_date);
				else
					insert into eventstemp(id, title , summary , category , image , update_date) values (rec.id, rec.title,rec.summary,rec.parentcategory , rec.image , rec.update_date);
				end if;
		end if;
			
		end loop;
		if array_length(categories,1)>=1 then 
			return query 
			select * from eventstemp where eventstemp.category = any(categories) order by eventstemp.update_date DESC;
		else
			return query
			select * from eventstemp order by eventstemp.update_date DESC;
		end if;
		
		
	END;
$function$
;

CREATE OR REPLACE FUNCTION public.find_bro_incomplete(node_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$

                        declare
                            id_bro int;

begin

                            
                        select
	id
into
	id_bro
from
	csf_manager_node
where
	parent_id = (
	select
		parent_id
	from
		csf_manager_node
	where
		id = node_id)
	and done = false
	and id != node_id
limit 1 ;
if not found then
return 0 ;
else
return id_bro ;
end if;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_bro_incomplete_purpose(_purpose_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$

declare

	parent_node_id int4;
	node_id_ int4;
	rec record ;
	id_bro int4 ;
	list_nodes_bro int[];
begin
	
	
select node_id  into node_id_ from csf_manager_purposenode where id=_purpose_id;   
raise notice 'node_id  %',node_id_;
select parent_id  into parent_node_id from csf_manager_node   where id=node_id_;  
raise notice 'parent_node_id  %',parent_node_id;
list_nodes_bro := array(select id from csf_manager_node  where parent_id =parent_node_id and id != node_id_) ;

select id

into
	id_bro
	
	from csf_manager_purposenode where node_id =any(list_nodes_bro) and done =false 

limit 1 ;
	




if not found then
return 0 ;
else

return id_bro ;
end if;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_category(list_event_id integer[])
 RETURNS SETOF threats_category
 LANGUAGE plpgsql
AS $function$
declare 
threats_scope_subcsfratevalue_id integer[] ;
threats_subcsfratevalue integer[] ;
begin    

threats_scope_subcsfratevalue_id := array(
select
	threats_scope.subcsfratevalue_id
from
	threats_scope
inner join threats_event_event_scope
    on
	(threats_scope.id = threats_event_event_scope.scope_id)
inner join threats_subcsfratevalue
    on
	(threats_scope.subcsfratevalue_id = threats_subcsfratevalue.id)
where
	(threats_event_event_scope.event_id = any (list_event_id)
		and threats_subcsfratevalue.category_id is not null)
	) ;

threats_subcsfratevalue := array(
SELECT 
       threats_subcsfratevalue.category_id
  FROM threats_subcsfratevalue
 WHERE threats_subcsfratevalue.id = any (threats_scope_subcsfratevalue_id)
) ;

return QUERY
select
	*
from
	threats_category
 WHERE threats_category.id = any (threats_subcsfratevalue)
;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_child(value_parent_id integer)
 RETURNS TABLE(csf_manager_node_id integer, csf_manager_node_event_effect_percent smallint)
 LANGUAGE plpgsql
AS $function$
                            declare
                            rec record ;

begin
                            return query
                            select
	id ,
	event_effect_percent
from
	csf_manager_node
where
	parent_id = value_parent_id ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_events(node_id integer)
 RETURNS integer[]
 LANGUAGE plpgsql
AS $function$
declare 
list_events int[];

list_events_final int[];

list_csf int[];

list_total int[];

record_child record ;

total_not_empty boolean := true ;

begin
list_total = array_append(list_total, node_id) ;

while total_not_empty loop
                            
                            
for record_child in

select
	id ,
	event_effect_percent ,
	csf_id,
	children_probably_percent
from
	csf_manager_node
where
	parent_id = node_id 

union 

select 

cmn.id,
cmn.event_effect_percent,
cmn.csf_id,
cmn.children_probably_percent

from 

csf_manager_treesrelation cmt 

join csf_manager_node cmn on

cmt.source_node_id=cmn.id

where destination_node_id=node_id

loop
if record_child.children_probably_percent <> 0 then
list_total = array_append(list_total, record_child.id);
end if ;

if record_child.event_effect_percent <> 0 then
list_events = array_append(list_events, record_child.csf_id);
end if ;
end loop;

list_csf = array_append(list_csf, node_id);

list_total := array_remove(list_total , node_id );

if list_total[1] is not null then
                            
node_id = list_total[1] ;
else
exit ;
end if ;
end loop ;

list_events_final := array(
select
	distinct 
	id
from
	threats_event
where
	csf_id = any(list_events)

) ;

return list_events_final;
end;

$function$
;


CREATE OR REPLACE FUNCTION public.find_leaf()
 RETURNS integer
 LANGUAGE plpgsql
AS $function$
                        declare
                            rec record;

node_id int ;

begin

                                select
	*
into
	rec
from
	csf_manager_node t_nod1
left outer join csf_manager_node t_nod2 on
	(t_nod1."id" = t_nod2."parent_id")
where
	(
                        t_nod2."id" is null
		and t_nod1."done" = false 
                        )
limit 1 ;

if rec.id > 0 then

                            return rec.id ;
end if;

return 0;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_leaf_purpose(_purpose_title integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$
                        declare
                            



list_nodes int[];
child int4 ;
begin

list_nodes := array(
select
	t_nod1.id
from
	csf_manager_node t_nod1
left outer join csf_manager_node t_nod2 on
	(t_nod1."id" = t_nod2."parent_id")
where
	t_nod2."id" is null);

select id into child from csf_manager_purposenode where done =false and node_id =any(list_nodes) and purpose_id =_purpose_title limit 1 ;
if child > 0 then

return child ;
end if;

return 0;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_nodes_by_event(event_ids integer)
 RETURNS integer[]
 LANGUAGE plpgsql
AS $function$
declare 

list_nodes int[];

begin
	list_nodes := array(
select
	node_id
from
	csf_manager_node_event
where
	event_id = event_ids
	);
return list_nodes;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_parent(node_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$
declare 
parent_id_reult int ;

begin
	
select
	parent_id
into
	parent_id_reult
from
	csf_manager_node
where
	id = node_id ;

if parent_id_reult > 0 then

return parent_id_reult ;
end if;

return 0 ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_purpose_parent(_purpose_id integer)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$
declare 
parent_id_reult int ;

begin
select id
into
	parent_id_reult
	from csf_manager_purposenode cmp2 where node_id =	(
select
	cmn.parent_id

from
	csf_manager_node cmn join csf_manager_purposenode cmp on cmn.id = cmp.node_id  
where
	cmp.id = _purpose_id );

if parent_id_reult > 0 then

return parent_id_reult ;
end if;

return 0 ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_step_parent(node_id integer)
 RETURNS integer[]
 LANGUAGE plpgsql
AS $function$
declare 
list_step_parent int[] ;

begin
	
list_step_parent := array(
select
	destination_node_id
from
	csf_manager_treesrelation
where
	source_node_id = node_id ) ;

return list_step_parent ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.find_tnapcy(list_event_id integer[])
 RETURNS integer[]
 LANGUAGE plpgsql
AS $function$
declare 
list_tnapcy integer[] ;

begin    

list_tnapcy := array(
select
	distinct 
tnapcy_id
from
	threats_threatsevent2
where
	event_id = any (list_event_id)
	
	) ;

return list_tnapcy;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.function_severity_expire_geometric_reduction(node_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare              
rec record ;

is_expire bool := false ;

Remainder numeric(7,4) ;

begin
        select
	expire_date ,
	DATE(update_date) as update_date ,
	severity
into
	rec
from
	threats_event
where
	id = node_id ;

Remainder = 1 / ((( (CURRENT_DATE - rec.update_date)::numeric(7,4) / (rec.expire_date - rec.update_date)::numeric(7,4) ) * 2.7 ) + 0.3);

Remainder = (Remainder * rec.severity) / 3 ;

if (rec.expire_date - CURRENT_DATE) <= 0
	then
	is_expire = true ;

Remainder = 0 ;
end if ;

if Remainder > rec.severity then Remainder = rec.severity ;
end if ;
update
	threats_event
set
	is_expired = is_expire ,
	severity_according_expire_date = Remainder
where
	id = node_id ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.function_severity_expire_linear_incremental(node_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare              
rec record ;

is_expire bool := false ;

Remainder numeric(5,
2) ;

expire_date_val date ;

update_date_val date ;

begin
	
	        select
	expire_date ,
	DATE(update_date) as update_date ,
	severity
into
	rec
from
	threats_event
where
	id = node_id ;

if (rec.expire_date - CURRENT_DATE) <= 0
	then
	is_expire = true ;

Remainder = 0 ;
else



Remainder =  ((CURRENT_DATE - rec.update_date)::numeric(3,
0) / (rec.expire_date - rec.update_date)::numeric(3,
0)) ;

end if ;

update
	threats_event
set
	is_expired = is_expire ,
	severity_according_expire_date =Remainder * rec.severity
where
	id = node_id ;
end;

$function$
;


CREATE OR REPLACE FUNCTION public.function_severity_expire_linear_reduction(node_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare              
rec record ;

is_expire bool := false ;

Remainder numeric(7,4) ;

expire_date_val date ;

update_date_val date ;

begin
	
	        select
	expire_date ,
	DATE(update_date) as update_date ,
	severity
into
	rec
from
	threats_event
where
	id = node_id ;

if (rec.expire_date - CURRENT_DATE) <= 0
	then
	is_expire = true ;

Remainder = 0 ;
else



Remainder = ((1 - ((CURRENT_DATE - rec.update_date)::numeric(7,4) / (rec.expire_date - rec.update_date)::numeric(7,4))) * rec.severity)/ 100 ;
end if ;

update
	threats_event
set
	is_expired = is_expire ,
	severity_according_expire_date =(Remainder * 100)
where
	id = node_id ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.function_severity_expire_stable(node_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare              
rec record ;

begin
        select
	expire_date ,
	severity_according_expire_date,
	severity
	
into
	rec
from
	threats_event
where
	id = node_id ;

if (rec.expire_date - CURRENT_DATE) <= 0
	then
update
	threats_event
set
	is_expired = true ,
	severity_according_expire_date = 0
where
	id = node_id ;
else

update
	threats_event
set
	
	severity_according_expire_date = rec.severity
where
	id = node_id ;
end if ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_check_revolve_cluster(source_head_id integer, destination_head_id integer)
 RETURNS boolean
 LANGUAGE plpgsql
AS $function$
declare 
list_dest int[];

list_head_source int[];

head int ;

list_total int[];

record_child record ;

total_not_empty boolean := true ;

begin
list_total = array_append(list_total, destination_head_id) ;

while total_not_empty loop

if list_total[1] is not null then
                            
head = list_total[1] ;
else
total_not_empty = false ;

exit ;
end if ;

list_head_source = array(
select
	csf.head_id
from
	csf_manager_node csf
where
	id = any(
	select
		csf_manager_treesrelation.destination_node_id
	from
		csf_manager_node
	join csf_manager_treesrelation on
		csf_manager_treesrelation.source_node_id = csf_manager_node.id
	where
		csf_manager_node.head_id = head
	)
		) ;

list_total := array_remove(list_total , head );

list_total = array_cat(list_total , list_head_source);

if (
select
	source_head_id = any (list_total) ) then

return true ;

exit ;
end if ;
end loop ;

return false ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_compute_importance()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
                        declare 
                        counter real = 0;

scan real = 0 ;

total real ;

list_parents int[] ;

has_leaf boolean = true ;

progress_to_root boolean := true ;

parent_id int ;

node_id int ;

begin
                        perform public.set_empty_importance_percent();

while has_leaf loop

                        select
	public.find_leaf()
                        into
	node_id ;

if node_id = 0 then
                        has_leaf := false ;

raise notice 'leaf not found';
else

                        while progress_to_root loop
                        list_parents = array_append(list_parents, node_id);

counter = counter + 1 ;

update
	csf_manager_node
set
	done = true
where
	id = node_id ;

select
	public.find_parent(node_id)
                        into
	parent_id;

if parent_id = 0 then
                        while counter >= scan loop


                        total = 1 - (scan / counter);

update
	csf_manager_node
set
	importance_percent = (total * 90)
where
	id = list_parents[array_length(list_parents, 1)]
	and importance_percent <= (total * 90) ;

list_parents := array_remove(list_parents, list_parents[array_length(list_parents, 1)]);

scan = scan + 1 ;
end loop ;

counter = 0;

scan = 0;

exit ;
else
                        node_id = parent_id ;
end if ;
end loop ;
end if;
end loop;

perform public.refresh_tree();
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_computed_actor_keyword_category(node_id integer, actor_ref refcursor, keyword_ref refcursor, category_ref refcursor)
 RETURNS SETOF refcursor
 LANGUAGE plpgsql
AS $function$

declare 

list_events int[];

list_tnapcy int[];

rec_category record ;

begin
	
select
	public.find_events(node_id)
into
	list_events;

if array_length(list_events, 1) > 0 then

list_tnapcy := array(
select
	public.find_tnapcy(list_events)
);
end if ;

open actor_ref for
select
	distinct threats_actor.name ,
	threats_tnapcy_actors.actor_id
from
	threats_tnapcy_actors
inner join threats_actor on
	threats_tnapcy_actors.actor_id = threats_actor.id
where
	tnapcy_id = any (list_tnapcy);

return next actor_ref;

open keyword_ref for
select
	distinct threats_kyword.name ,
	threats_tnapcy_keywords.kyword_id
from
	threats_tnapcy_keywords
inner join threats_kyword on
	threats_tnapcy_keywords.kyword_id = threats_kyword.id
where
	tnapcy_id = any (list_tnapcy);

return next keyword_ref;

open category_ref for
select
	*
from
	public.find_category(list_events) ;

return next category_ref;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_computed_events(node_id integer, page_num integer DEFAULT 1, limit_num integer DEFAULT 5)
 RETURNS SETOF threats_event
 LANGUAGE plpgsql
AS $function$
declare 
list_events int[];

list_csf int[];

list_total int[];

record_child record ;

total_not_empty boolean := true ;

offset_num integer := 0 ;

count_event_limit integer:= 0 ;


begin
	
offset_num = limit_num * (page_num - 1) ;


list_total = array_append(list_total, node_id) ;

while total_not_empty loop
                            

for record_child in

select
	id ,
	event_effect_percent ,
	csf_id,
	children_probably_percent
from
	csf_manager_node
where
	parent_id = node_id 
	
union 

select 

cmn.id,
cmn.event_effect_percent,
cmn.csf_id,
cmn.children_probably_percent

from 

csf_manager_treesrelation cmt 

join csf_manager_node cmn on

cmt.source_node_id=cmn.id

where destination_node_id=node_id


loop
if record_child.children_probably_percent <> 0 then
list_total = array_append(list_total, record_child.id);
end if ;

if record_child.event_effect_percent <> 0 then
                                list_events = array_append(list_events, record_child.csf_id);
                               count_event_limit = count_event_limit +1 ;
                              raise notice 'count  %', count_event_limit;
end if ;
end loop;

list_csf = array_append(list_csf, node_id);

list_total := array_remove(list_total , node_id );

if list_total[1] is not null then
                            
node_id = list_total[1] ;

else

exit ;
end if ;

if count_event_limit > (limit_num *  page_num) then 
raise notice 'gg %', count_event_limit;
exit ;
end if ;

end loop ;



return QUERY
select
	*
from
	threats_event t
where
	csf_id = any(list_events) and expire_date > current_date 
order by
	update_date desc
limit limit_num offset offset_num
	;
end;

$function$
;


CREATE OR REPLACE FUNCTION public.main_computed_severity_improved(node_list integer[], event_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$

declare 
progress_to_root boolean := true ;

node_id integer := 0 ;

begin

foreach node_id in array node_list

loop
  progress_to_root = true ;

while progress_to_root loop

raise notice 'node_id %',
node_id ;

if node_id = 0 then
progress_to_root := false ;
else

perform public.update_node(node_id ,
event_id);
end if;

select
	public.find_parent(node_id)
into
	node_id;
end loop;
end loop;

perform public.refresh_tree();
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_computed_severity_purpose_improved(_purpose_list integer[], _event_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$

declare 
progress_to_root boolean := true ;

purpose_id integer := 0 ;

begin

foreach purpose_id in array _purpose_list

loop
  progress_to_root = true ;

while progress_to_root loop

raise notice 'purpose_id %',
purpose_id ;

if purpose_id = 0 then
progress_to_root := false ;
else

perform public.update_purpose(purpose_id ,
_event_id);
end if;

select
	public.find_purpose_parent(purpose_id)
into
	purpose_id;
end loop;
end loop;

perform public.refresh_purpose(0);
end;

$function$
;


CREATE OR REPLACE FUNCTION public.main_computed_sub_csf(node_id integer)
 RETURNS TABLE(pk integer, events numeric(7,4), score smallint, child numeric(7,4), parent integer, csf integer, title text)
 LANGUAGE plpgsql
AS $function$
declare 
list_events int[];

list_csf int[];

list_total int[];

record_child record ;

total_not_empty boolean := true ;

begin
list_total = array_append(list_total, node_id) ;

while total_not_empty loop
                            
                            
for record_child in

select
	id ,
	event_effect_percent ,
	csf_id,
	children_probably_percent
from
	csf_manager_node
where
	parent_id = node_id 
loop
if record_child.children_probably_percent <> 0 then
list_total = array_append(list_total, record_child.id);
end if ;

if record_child.event_effect_percent <> 0 then
list_events = array_append(list_events, record_child.id);
end if ;
end loop;

list_csf = array_append(list_csf, node_id);

list_total := array_remove(list_total , node_id );

if list_total[1] is not null then
                            
node_id = list_total[1] ;
else

exit ;
end if ;
end loop ;

list_csf = array_cat(list_csf , list_events);

return QUERY
select
	csf_manager_node.id ,
	csf_manager_node.event_effect_percent,
	csf_manager_node.score ,
	csf_manager_node.children_probably_percent ,
	csf_manager_node.parent_id ,
	csf_manager_node.csf_id,
	threats_csf.csf_title
from
	csf_manager_node
inner join threats_csf on
	(csf_manager_node.csf_id = threats_csf.id)
where
	csf_manager_node.id = any(list_csf) ;
end;

$function$
;


CREATE OR REPLACE FUNCTION public.main_computed_tree_by_list_node(event_ids integer, list_node integer[])
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare
rec record;
type_expire_dates smallint ;
node int;
begin
	
	
	 select
	
	type_expire_date into type_expire_dates
from
	threats_event
where
	id = event_ids ;
	
perform public.specifies_type_expire_date(event_ids,type_expire_dates);

perform public.compute_total_severity(list_node);

perform public.main_computed_severity_improved(list_node,
event_ids);
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_computed_tree_by_list_purpose(_event_id integer, _list_purpose integer[])
 RETURNS void
 LANGUAGE plpgsql
AS $function$
declare
rec record;
type_expire_dates smallint ;

begin
	
	
	 select
	
	type_expire_date into type_expire_dates
from
	threats_event
where
	id = _event_id ;
	
perform public.specifies_type_expire_date(_event_id,type_expire_dates);

perform public.compute_total_purpose_severity(_list_purpose);

perform public.main_computed_severity_purpose_improved(_list_purpose,
_event_id);
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_find_child_node_events(value_parent_id integer, node_child refcursor, events refcursor)
 RETURNS SETOF refcursor
 LANGUAGE plpgsql
AS $function$
declare
childs record ;

begin
	
drop table if exists temp_events;

create temporary table temp_events(
    
 id serial4 not null,
 title varchar(1000) null,
 summary text null ,
 image varchar(100) null,
 create_date timestamptz not null,
 update_date timestamptz not null,
 severity int2 null,
 expire_date date null ,
 is_expired bool not null ,
 severity_according_expire_date numeric(7,4) ,
 importance int2 null ,
 csf_id int 
);

drop table if exists temp_node_child;

create temporary table temp_node_child(
 id serial4 not null,
 csf_id int ,
 probability numeric(7,4) not null,
 event_effect_percent numeric(7,4) not null,
 csf_title text not null
  
);

insert
	into
	temp_node_child (id,
	csf_id,
	probability,
	event_effect_percent,
	csf_title
	)

select
	csf_manager_node.id as ids,
	csf_manager_node.csf_id as csf_id,
	csf_manager_node.probability as probabilitys,
	csf_manager_node.event_effect_percent as event_effect_percent,
	threats_csf .csf_title as titles
from
	csf_manager_node
join threats_csf on
	csf_manager_node.csf_id = threats_csf .id
where
	csf_manager_node.parent_id = value_parent_id
	and csf_manager_node.probability > 0 ;

open node_child for
select
	*
from
	temp_node_child;

return next node_child;

for childs in
	
select
	id ,
	csf_id
from
	temp_node_child
loop
	
	insert
	into
	temp_events (id,
	title,
	summary,
	image,
	create_date,
	update_date,
	severity,
	expire_date,
	is_expired,
	severity_according_expire_date,
	importance ,
	csf_id 
	)
select
	id,
	title,
	summary,
	image,
	create_date,
	update_date,
	severity,
	expire_date,
	is_expired,
	severity_according_expire_date,
	importance,
	childs.id
from
	public.threats_event
where
	csf_id = childs.csf_id
	and is_expired is false
order by
	create_date desc

;

insert
	into
	temp_events (id,
	title,
	summary,
	image,
	create_date,
	update_date,
	severity,
	expire_date,
	is_expired,
	severity_according_expire_date,
	importance ,
	csf_id 
	)
select
	id,
	title,
	summary,
	image,
	create_date,
	update_date,
	severity,
	expire_date,
	is_expired,
	severity_according_expire_date,
	importance,
	childs.id
from
	public.main_computed_events(childs.id,
	1,
	5)
;
end loop;

open events for
select
	*
from
	temp_events;

return next events;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_restart_severity()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
-- // if  event_id = 0 then compude whitout update history threat 
declare 
    progress_to_root boolean := true ;

has_bro boolean := true ;

node_id integer ;

parent_id integer ;

bro_id integer ;

check_children_done_boolean boolean ;

list_nodes int[];

begin
	
list_nodes := array(select id from csf_manager_node );

--raise notice 'list nodes %' , list_nodes ;

perform public.compute_total_severity(list_nodes);

while progress_to_root loop

select
	public.find_leaf()
into
	node_id ;

if node_id = 0 then
progress_to_root := false ;

--raise notice 'leaf not found';
else
perform public.update_node(node_id , 0);


end if;

--raise notice 'leaf found %',
--node_id;

has_bro := true ;

while has_bro loop
                        select
	public.find_bro_incomplete(node_id)
                        into
	bro_id ;

if bro_id = 0 then
                        select
	public.find_parent(node_id)
into
	parent_id;

node_id = parent_id ;

perform public.update_node(node_id , 0);

--raise notice 'find parent and update';
if node_id = 0 then
exit;
end if;
else 
                        select
	public.check_children_done(bro_id)
                        into
	check_children_done_boolean;

if check_children_done_boolean then
                        node_id = bro_id ;

perform public.update_node(node_id , 0);

--raise notice 'update_node %',
--                        node_id;
else
                            has_bro := false ;

--raise notice 'child_incomplete';
end if;
end if ;
end loop;
end loop;

perform public.refresh_tree();
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_restart_severity_purpose(_purpose_title integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
-- // if  event_id = 0 then compude whitout update history threat 
declare 

progress_to_root boolean := true ;
has_bro boolean := true ;
node_id integer ;
purpose_id_ integer ;
parent_id integer ;
bro_id integer ;
check_children_done_boolean boolean ;

list_purpose int[];

begin
	
list_purpose := array(select id from csf_manager_purposenode where purpose_id =_purpose_title );

--raise notice 'list nodes %' , list_nodes ;

perform public.compute_total_purpose_severity(list_purpose);

while progress_to_root loop

select
	public.find_leaf_purpose(_purpose_title)
into
	purpose_id_ ;

if purpose_id_ = 0 then
progress_to_root := false ;

--raise notice 'leaf not found';
else
perform public.update_purpose(purpose_id_ , 0);


end if;

--raise notice 'leaf found %',
--node_id;

has_bro := true ;

while has_bro loop
                        select
	public.find_bro_incomplete_purpose(purpose_id_)
                        into
	bro_id ;

if bro_id = 0 then
raise notice 'bro_id is 0';
                        select
	public.find_purpose_parent(purpose_id_)
into
	parent_id;

purpose_id_ = parent_id ;

perform public.update_purpose(purpose_id_ , 0);

--raise notice 'find parent and update';
if purpose_id_ = 0 then
exit;
end if;
else 
raise notice 'bro_id is %',bro_id;
select
	public.check_children_done_purpose(bro_id)
                        into
	check_children_done_boolean;

if check_children_done_boolean then
                        purpose_id_ = bro_id ;

perform public.update_purpose(purpose_id_ , 0);

--raise notice 'update_node %',
--                        node_id;
else
                            has_bro := false ;

--raise notice 'child_incomplete';
end if;
end if ;
end loop;
end loop;

perform public.refresh_purpose(_purpose_title);
end;

$function$
;

CREATE OR REPLACE FUNCTION public.main_update_severity_according_expire_date()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
								
declare 
rec record;

begin
	
	 for rec in
      select
	id ,
	type_expire_date
from
	threats_event
where
	is_expired is false 

      
   loop
perform public.specifies_type_expire_date(rec.id,rec.type_expire_date) ;
end loop;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.purpose__csf_node_to_purpose()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
	BEGIN

insert
	into
	csf_manager_purposenode (
	score,
	event_effect_percent,
	children_probably_percent,
	probability,
	positive,
	updated_probability,
	node_id,
	done,
	purpose_id
	)

select score ,event_effect_percent ,children_probably_percent ,probability ,positive ,updated_probability ,id ,false,1 from csf_manager_node cmn where cluster_id =20 ;  
	END;
$function$
;

CREATE OR REPLACE FUNCTION public.purpose__node_event_to_purpose_event()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
	BEGIN

		
		
		
		
		
		
insert
	into
	csf_manager_purposenode_event  (
		purposenode_id ,
		event_id 
	)

select  cmp.id as p , cmne.event_id  as e from csf_manager_node_event cmne join csf_manager_purposenode cmp on cmne.node_id = cmp.node_id  ;  
	END;
$function$
;

CREATE OR REPLACE FUNCTION public.refresh_purpose(_purpose_title integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
                            begin

                                update
	csf_manager_purposenode 
set
	done = false where 
	case 
	when _purpose_title = 0 then 
	
	true
	
	when _purpose_title <> 0 then 
	
	purpose_id =  _purpose_title
	
	end
;

raise notice 'set false done tree  ' ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.refresh_tree()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
                            begin

                                update
	csf_manager_node
set
	done = false;

raise notice 'set false done tree  ' ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.refresh_tree_purpose(_purpose_title integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
                            begin

                                update
	csf_manager_purposenode 
set
	done = false where purpose_id =_purpose_title ;

--raise notice 'set false done tree  ' ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.set_empty_importance_percent()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
                            begin
                        update
	csf_manager_node
set
	importance_percent = 5 ,
	done = false ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.set_history_threats(severity_val numeric(7,4), event_val numeric(7,4), threat_val integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$

declare 
old_severity smallint;

begin            

if (event_val <> 0)
and (threat_val <> 0) then    

select

	   severity   

	   into
	old_severity
from
	csf_manager_historythreat
where
	threat_id = threat_val
order by
	id desc
limit 1;

if ( old_severity <> severity_val )
or ( old_severity is null
	and severity_val <> 0 ) then 
		insert
	into
	csf_manager_historythreat (severity,
	event_id ,
	threat_id,
	date)
values(severity_val,
event_val,
threat_val,
current_timestamp);
end if ;
end if ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.specifies_type_expire_date(event_id integer, type_expire_date smallint)
 RETURNS void
 LANGUAGE plpgsql
AS $function$

begin
	

 	if type_expire_date = 2 then
 	 
 	perform public.function_severity_expire_linear_reduction(event_id);

elsif type_expire_date = 1 then

perform public.function_severity_expire_stable(event_id);

elsif type_expire_date = 3 then

perform public.function_severity_expire_geometric_reduction(event_id);

elsif type_expire_date = 4 then

perform public.function_severity_expire_linear_Incremental(event_id);
end if ;

end;

$function$
;

CREATE OR REPLACE FUNCTION public.test1(source_head_id integer, destination_head_id integer)
 RETURNS boolean
 LANGUAGE plpgsql
AS $function$
declare 
list_dest int[];

list_head_source int[];

head int ;

list_total int[];

record_child record ;

total_not_empty boolean := true ;

begin
list_total = array_append(list_total, destination_head_id) ;

while total_not_empty loop

if list_total[1] is not null then
                            
head = list_total[1] ;
else
total_not_empty = false ;

exit ;
end if ;

list_head_source = array(
select
	csf.head_id
from
	csf_manager_node csf
where
	id = any(
	select
		csf_manager_treesrelation.destination_node_id
	from
		csf_manager_node
	join csf_manager_treesrelation on
		csf_manager_treesrelation.source_node_id = csf_manager_node.id
	where
		csf_manager_node.head_id = head
	)
		) ;

list_total := array_remove(list_total , head );

list_total = array_cat(list_total , list_head_source);

if (
select
	source_head_id = any (list_total) ) then

return true ;

exit ;
end if ;
end loop ;

return false ;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.trigger_main_computed_importance()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
begin
if TG_OP = 'UPDATE' then
if NEW.parent_id <> old.parent_id then
perform public.main_compute_importance();

return new;
end if ;
end if ;

if TG_OP = 'DELETE' then

perform public.main_compute_importance();

return old;
end if ;

if TG_OP = 'INSERT' then

perform public.main_compute_importance();

return new;
end if ;

return new;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.trigger_update_event_severity()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
declare
list_nodes int[];

begin
if TG_OP = 'DELETE' then

list_nodes := array(
select
	find_nodes_by_event(old.id)) ;

perform public.compute_total_severity(list_nodes);

raise notice 'finish' ;

perform public.main_computed_severity_improved(list_nodes,
0);

return old;
end if ;

if TG_OP = 'UPDATE' then
if NEW.csf_id <> OLD.csf_id then
list_nodes := array(
select
	find_nodes_by_event(old.id)) ;

perform public.compute_total_severity(list_nodes);
end if;

if (NEW.severity <> OLD.severity)
or ( (OLD.severity is null)
and (new.severity is not null))
or (NEW.csf_id <> OLD.csf_id) then
list_nodes := array(
select
	find_nodes_by_event(new.id)) ;

perform public.compute_total_severity(list_nodes);

raise notice 'finish' ;

perform public.main_computed_severity_improved(new.csf_id,
new.id);

return new;
end if;
end if;

if TG_OP = 'INSERT' then        

list_nodes := array(
select
	find_nodes_by_event(new.id)) ;

perform public.compute_total_severity(list_nodes);

perform public.main_computed_severity_improved(new.csf_id,
new.id);

return new;
end if;

return new;
end;

$function$
;

CREATE OR REPLACE FUNCTION public.update_node(node_id integer, event_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$

declare 
result_child numeric(7,4) = 0;

Remainder numeric(7,4) = 0;

rec record ;

begin
                            
select
	*
into
	rec
from
	csf_manager_node
where
	id = node_id ;

Remainder = 100 - rec.event_effect_percent ;

select
	public.compute_child(node_id)
into
	result_child;
if result_child is null then 
--	raise notice 'iffffff' ;
	result_child = 0 ;
end if ;
--raise notice 'type % ggg %' ,  pg_typeof(result_child) , result_child ;



update
	csf_manager_node
set
	done = true,
	children_probably_percent = result_child ,
	probability = (( (result_child * Remainder) / 100) + event_effect_percent)
where
	id = node_id;

perform public.set_history_threats(result_child , event_id , node_id ) ; 
end;

$function$
;
CREATE OR REPLACE FUNCTION public.is_same_csf_in_parents_for_one_node(node_id integer, _csf_id integer)
 RETURNS boolean
 LANGUAGE plpgsql
AS $function$

declare 
	has_same_csf bool :=false ;
	has_parent bool :=true;
	node record;
	stat_parent_id int :=node_id ;
	stat_csf_id int:=_csf_id;

begin 
	
	while has_parent loop
		
		
		for node in select csf_id,parent_id from csf_manager_node where id=stat_parent_id loop 
			
			if node.parent_id is not null then
				
				stat_parent_id := node.parent_id;
				raise notice 'parent id : % -------------(stat_id : %)',node.parent_id, stat_parent_id;
			
			elsif node.parent_id is null then
				has_parent :=false;
			end if;
		
			if node.csf_id = _csf_id then
				has_same_csf := true;
			end if;
		
		end loop;
		
	end loop;
	
	return has_same_csf;
	
end;

$function$
;

CREATE OR REPLACE FUNCTION public.update_purpose(_purpose_id integer, _event_id integer)
 RETURNS void
 LANGUAGE plpgsql
AS $function$

declare 
result_child smallint = 0;

Remainder smallint = 0;

rec record ;

begin
                            
select
	*
into
	rec
from
	csf_manager_purposenode cmp 
where
	id = _purpose_id ;

Remainder = 100 - rec.event_effect_percent ;

raise notice 'update_purpose % -----% ',_purpose_id,Remainder ;
select
	public.compute_purpose_child(_purpose_id)
into
	result_child;
if result_child is null then 
--	raise notice 'iffffff' ;
	result_child = 0 ;
end if ;
--raise notice 'type % ggg %' ,  pg_typeof(result_child) , result_child ;



update
	csf_manager_purposenode 
set
	done = true,
	children_probably_percent = result_child ,
	probability = (( (result_child * Remainder) / 100) + event_effect_percent)
where
	id = _purpose_id;

select probability into Remainder from csf_manager_purposenode where id = _purpose_id;

raise notice 'finish --- > %',Remainder;
perform public.set_history_threats(result_child , _event_id , _purpose_id ) ; 
end;

$function$
;

                    '''
                    )

                self.stdout.write("\n\ncomplete create_function  \n\n" )
            

                     
        if type == 'remove_function':


            with connection.cursor() as cursor:
        
                cursor.execute(

                    '''
                        DROP FUNCTION  IF EXISTS public.check_children_done(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.check_have_bro_incomplete_or_id_bro(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.compute_child(value_parent_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.compute_total_severity(_int4) CASCADE;
                        DROP FUNCTION  IF EXISTS filter_event_except_category(varchar,varchar[],integer[],integer,date,date,varchar[],varchar[],integer[],boolean);
                        DROP FUNCTION  IF EXISTS filter_event_function(varchar,varchar[],integer[],integer,date,date,varchar[],varchar[],integer[],varchar[],boolean);
                        DROP FUNCTION  IF EXISTS public.find_bro_incomplete(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.find_category(list_event_id integer[]) CASCADE;              
                        DROP FUNCTION  IF EXISTS public.find_events(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.find_leaf() CASCADE;
                        DROP FUNCTION  IF EXISTS public.find_nodes_by_event(event_ids integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.find_parent(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.find_tnapcy(list_event_id integer[]) CASCADE;
                        DROP FUNCTION  IF EXISTS public.function_severity_expire_geometric_reduction(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.function_severity_expire_linear_incremental(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.function_severity_expire_linear_reduction(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.function_severity_expire_stable(node_id integer) CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_compute_importance() CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_computed_actor_keyword_category(node_id integer, actor_ref refcursor, keyword_ref refcursor, category_ref refcursor) CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_computed_events(int4, int4 , int4) CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_restart_severity() CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_update_severity_according_expire_date() CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_computed_severity_improved(_int4, int4) CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_computed_sub_csf(node_id integer) CASCADE;		
                        DROP FUNCTION  IF EXISTS public.main_computed_tree_by_list_node(int4, _int4) CASCADE;
                        DROP FUNCTION  IF EXISTS public.main_find_child_node_events(value_parent_id integer, node_child refcursor, events refcursor) CASCADE;
                        DROP FUNCTION  IF EXISTS public.refresh_tree() CASCADE;
                        DROP FUNCTION  IF EXISTS public.set_empty_importance_percent() CASCADE;
                        DROP FUNCTION  IF EXISTS public.set_history_threats(severity_val smallint, event_val integer, threat_val integer) CASCADE;      
                        DROP FUNCTION  IF EXISTS public.specifies_type_expire_date(event_id integer, type_expire_date smallint)  CASCADE;      
                        DROP FUNCTION  IF EXISTS public.trigger_main_computed_importance() CASCADE;
                        DROP FUNCTION  IF EXISTS public.trigger_update_event_severity() CASCADE;
                        DROP FUNCTION  IF EXISTS public.update_node(int4,int4) CASCADE;
                        DROP FUNCTION  IF EXISTS public.is_same_csf_in_parents_for_one_node(int4,int4) CASCADE;
					'''
                    )

                self.stdout.write("\n\ncomplete remove_function  \n\n" )
            


        if type == 'create_trigger':


            with connection.cursor() as cursor:
        
                cursor.execute(

                    '''
create trigger trigger_main_computed_importance after
insert
	or
delete
	or
update
	on
	public.csf_manager_node for each row execute PROCEDURE trigger_main_computed_importance() ;


