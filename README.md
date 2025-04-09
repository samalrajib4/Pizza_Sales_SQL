Pizza_Sales_Project

-- Pizza_Sales_Project....


-- #Start
-- Retrieve the total number of orders placed.

select count(order_id) as Total_orders
from orders;

-- Calculate the total revenue generated from pizza sales.

select round(sum(price*quantity),2) as Total_sales
from order_details
left join pizzas
	on order_details.pizza_id = pizzas.pizza_id;
    
-- Identify the highest-priced pizza.

select pizza_types.`name`, pizzas.price
from pizza_types
join pizzas
	on pizza_types.pizza_type_id = pizzas.pizza_type_id
order by price desc limit 1;

-- Identify the most common pizza size ordered.

select pizzas.size, count(order_details.oredr_details_id) Frequently_Ordered_Pizza
from order_details
join pizzas
	on pizzas.pizza_id = order_details.pizza_id
group by pizzas.size
order by Frequently_Ordered_Pizza desc limit 1
;

-- List the top 5 most ordered pizza types along with their quantities.

select pizza_types.`name`, sum(order_details.quantity) as Quantities_Ordered
from pizza_types
join pizzas
	on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
	on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.`name`
order by Quantities_Ordered desc limit 5;

-- Find the total quantity of each pizza category ordered.

select pizza_types.category, sum(order_details.quantity) Quantity_Ordered
from pizza_types
join pizzas
	on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
	on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category
order by Quantity_Ordered desc;

-- Determine the distribution of orders by hour of the day.

select hour(order_time), count(order_id)
from orders
group by hour(order_time)
order by hour(order_time);

-- Find the category-wise distribution of pizzas.

select category, count(name)
from pizza_types
group by(category);

-- Group the orders by date and calculate the average number of pizzas ordered per day.

With CTE_Example as
(
select orders.order_date, sum(order_details.quantity) as quantity
from orders
join order_details
	on orders.order_id = order_details.order_id
group by orders.order_date
)
select round(avg(quantity),0) as Avg_Pizza_Ordered_per_day
from CTE_Example;

-- Determine the top 3 most ordered pizza types based on revenue.

select pizza_types.`name`, round(sum(order_details.quantity*pizzas.price),2) as Revenue_generated
from pizza_types
join pizzas
	on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
	on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.`name`
order by Revenue_generated desc
limit 3;
    
-- Calculate the percentage contribution of each pizza type to total revenue.

select pizza_types.category, 
round(sum(order_details.quantity*pizzas.price)/
	(select sum(price*quantity) 
	from order_details
	join pizzas
		on order_details.pizza_id = pizzas.pizza_id)*100,2) as Revenue_generated
from pizza_types
join pizzas
	on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
	on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category
order by Revenue_generated desc;

-- Analyze the cumulative revenue generated over time.

with CTE_Example as
(
select orders.order_date, round(sum(order_details.quantity*pizzas.price),2) as Revenue
from order_details
join pizzas
	on order_details.pizza_id = pizzas.pizza_id
join orders
	on orders.order_id = order_details.order_id
group by orders.order_date
)
select order_date,Revenue,
sum(Revenue) over (order by order_date) as Comulatyive_Revenue
from CTE_Example;

-- Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select `name`,revenue
from
	(select category,`name`,revenue,
	rank() over(partition by category order by revenue desc) as rn
	from
		(select pizza_types.category,pizza_types.`name`,
        sum(order_details.quantity*pizzas.price) as Revenue
		from order_details
		join pizzas
			on order_details.pizza_id = pizzas.pizza_id
		join pizza_types
			on pizza_types.pizza_type_id = pizzas.pizza_type_id
		group by pizza_types.category, pizza_types.`name`) as A) as B
		where rn <= 3;



-- END OF THE PROJECT
-- THANK YOU
