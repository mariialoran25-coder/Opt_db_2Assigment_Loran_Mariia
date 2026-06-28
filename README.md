# Opt_db_2Assigment_Loran_Mariia

Я створила неоптимізований запит і нижче добавила 4 етапи того як я оптимізовувала неоптимізований код. Всі ці 4 етапи валідні вони показують одне і теж просто намагалась якомога більше оптимізувати і покращити читабельність коду.

Мій запит виводе 3 стовпця в якому 1 рядочок(через агрегатні функції). Воно виводе імя + cnt;    cnt - min, max, avg;

Воно фільтровано за особливим емейлом , щоб в кінці було '%@example.com' , а статус активний і дата замовлення більша за 18 липня 2024року. Груповане та посортоване. Підраховує кількість рядків що профільтровані . І врешті решт виводе що саме я шукаю, а саме мінімальне, максимальне та середнє значення, які замовила якась людина відповідно до фільтрації;

Для оптимізації, я використовувала індекси . Далі об'єднала код який постійно повторювався в неоптимізованому коді. Я робила за допомогою CTE  , винесла спершу те що шукаю , об'єднала всі таблички і профільтрувала їх . І також склеїла відповідного до того що показував неоптимізований код.

На додаткове завдання я зробила BEGIN ;

set local enable_hashjoin = off; -- Коли я з'єднувала великі таблиці Join то Postgres сам обирає алгоритм як це зробити швидко. А вот Хеш джоін це і є 1 із алгоритмів
-- ще тут можна спробувати enable_seqscan, в Postgres муситиме використовувати Index Scan



ЗА СКІЛЬКИ ЧАСУ ВИКОНУЄТЬСЯ
Неоптимізований
Planning Time: 4.787 ms
Execution Time: 2379.883 ms
Тут бачимо, що досить повільно працює кверя через те що БД змушений читати кожен рядок таблиці, що дуже повільно для великої кількості даних.
А ще проблема в тому, що код повторюється , не читабельний. Немає індексів , коли він вибирає дані проходиться по таблицях декілька разів.

З ДОДАТКОВИМ ЗАВДАННЯМ set local enable_hashjoin = off
Planning Time: 3.709 ms
Execution Time: 2630.941 ms

Я заблокувала використання Hash JOin , що  погіршує ситуацію, бо в планувальника лишається 2 варіанти які менш ефективні.Тому Execution Time працює повільніше 



Оптимізований 1 етап 
Planning Time: 1.445 ms
Execution Time: 572.893 ms

З ІНДЕКСАМИ
Planning Time: 0.940 ms
Execution Time: 485.370 ms
---

Оптимізований 2 етап 
Planning Time: 1.096 ms
Execution Time: 778.102 ms

З ІНДЕКСАМИ
Planning Time: 0.792 ms
Execution Time: 793.228 ms

----
Оптимізований 3 етап 
Planning Time: 0.851 ms
Execution Time: 755.363 ms

З ІНДЕКСАМИ
Planning Time: 0.837 ms
Execution Time: 660.216 ms
----

Оптимізований 4 етап 
Planning Time: 1.532 ms
Execution Time: 633.991 ms

З ІНДЕКСАМИ
Planning Time: 1.530 ms
Execution Time: 637.137 ms

Я продеменстувала 4 етапи. Можна вибрати якусь 1 кверю, але я хотіла продеменстувати як можна оптимізовувати код, який буде швидшим і покращує читабельність коду.
Без індексів найшвидше Planning Time реалізує -  на 4 етапі оптимізації кверя.
А стосовно Execution Time - то найкраще реалізовує Оптимізований на 1 етапі кверя.;

Одже найкращиий варіант оптимізованого коду це оптимізований код на 1 етапі, бо Execution Time найменше показує часу за скільки кверя запрацює, це найшвидший резуьтат серед усіх етапів. Це працює через просту структуру.
Він легко читається.


. За допомогою індексів я оптимізовую код , бо вже замість повного читання я створюд додатковий стовпець з індексами і шукаю по ньому, що  значно ефектинвішим спосібом. До того ж індекс у відсортованому вигляді що дозволяє не сортувати дані і шукати за діапазоном



ОСЬ ПОВНА ВЕРСІЯ , що воноо мені показало 
У неопримізованому коді
Result  (cost=6024824.07..6024824.08 rows=1 width=96) (actual time=2440.340..2441.687 rows=1 loops=1)
  InitPlan 2
    ->  Limit  (cost=4673.73..5967348.39 rows=1 width=32) (actual time=736.519..737.547 rows=1 loops=1)
          ->  Subquery Scan on sub2  (cost=4673.73..2236007669.46 rows=375 width=32) (actual time=736.514..737.540 rows=1 loops=1)
                Filter: (sub2.cnt = (SubPlan 1))
                ->  GroupAggregate  (cost=4673.73..63339.60 rows=75037 width=21) (actual time=216.430..216.552 rows=1 loops=1)
"                      Group Key: oc_1.name, op_1.product_name"
                      ->  Incremental Sort  (cost=4673.73..62026.46 rows=75037 width=13) (actual time=216.410..216.531 rows=2 loops=1)
"                            Sort Key: oc_1.name, op_1.product_name"
                            Presorted Key: oc_1.name
                            Full-sort Groups: 1  Sort Method: quicksort  Average Memory: 27kB  Peak Memory: 27kB
                            Pre-sorted Groups: 1  Sort Method: quicksort  Average Memory: 30kB  Peak Memory: 30kB
                            ->  Nested Loop  (cost=4591.52..58534.32 rows=75037 width=13) (actual time=117.930..215.261 rows=178 loops=1)
                                  ->  Nested Loop  (cost=4591.22..56627.67 rows=75037 width=11) (actual time=117.499..212.907 rows=178 loops=1)
                                        ->  Gather Merge  (cost=4590.80..6885.82 rows=20137 width=23) (actual time=116.578..116.812 rows=55 loops=1)
                                              Workers Planned: 1
                                              Workers Launched: 1
                                              ->  Sort  (cost=3590.79..3620.40 rows=11845 width=23) (actual time=56.205..56.247 rows=28 loops=2)
                                                    Sort Key: oc_1.name
                                                    Sort Method: quicksort  Memory: 1440kB
                                                    Worker 0:  Sort Method: quicksort  Memory: 25kB
                                                    ->  Parallel Seq Scan on opt_clients oc_1  (cost=0.00..2789.35 rows=11845 width=23) (actual time=0.211..32.128 rows=8357 loops=2)
                                                          Filter: (((email)::text ~~ '%@example.com'::text) AND ((status)::text = 'active'::text))
                                                          Rows Removed by Filter: 41643
                                        ->  Index Scan using idx_opt_orders_client_id on opt_orders oo_1  (cost=0.42..2.43 rows=4 width=20) (actual time=0.656..1.742 rows=3 loops=55)
                                              Index Cond: (client_id = oc_1.id)
                                              Filter: (order_date > '2024-08-18'::date)
                                              Rows Removed by Filter: 6
                                  ->  Memoize  (cost=0.30..0.32 rows=1 width=10) (actual time=0.010..0.010 rows=1 loops=178)
                                        Cache Key: oo_1.product_id
                                        Cache Mode: logical
                                        Hits: 93  Misses: 85  Evictions: 0  Overflows: 0  Memory Usage: 10kB
                                        ->  Index Scan using opt_products_pkey on opt_products op_1  (cost=0.29..0.31 rows=1 width=10) (actual time=0.014..0.014 rows=1 loops=85)
                                              Index Cond: (product_id = oo_1.product_id)
                SubPlan 1
                  ->  Aggregate  (cost=29797.87..29797.88 rows=1 width=524) (actual time=519.669..520.573 rows=1 loops=1)
                        ->  Finalize GroupAggregate  (cost=19797.75..28859.91 rows=75037 width=21) (actual time=448.343..518.533 rows=26583 loops=1)
"                              Group Key: oc.name, op.product_name"
                              ->  Gather Merge  (cost=19797.75..27640.56 rows=62530 width=21) (actual time=448.333..503.054 rows=39726 loops=1)
                                    Workers Planned: 2
                                    Workers Launched: 2
                                    ->  Partial GroupAggregate  (cost=18797.72..19423.02 rows=31265 width=21) (actual time=310.120..322.351 rows=13242 loops=3)
"                                          Group Key: oc.name, op.product_name"
                                          ->  Sort  (cost=18797.72..18875.89 rows=31265 width=13) (actual time=310.098..314.019 rows=20496 loops=3)
"                                                Sort Key: oc.name, op.product_name"
                                                Sort Method: quicksort  Memory: 1698kB
                                                Worker 0:  Sort Method: quicksort  Memory: 1492kB
                                                Worker 1:  Sort Method: quicksort  Memory: 710kB
                                                ->  Hash Join  (cost=3412.42..16463.44 rows=31265 width=13) (actual time=22.726..194.824 rows=20496 loops=3)
                                                      Hash Cond: (oo.product_id = op.product_id)
                                                      ->  Parallel Hash Join  (cost=2937.42..15906.33 rows=31265 width=11) (actual time=17.069..183.239 rows=20496 loops=3)
                                                            Hash Cond: (oo.client_id = oc.id)
                                                            ->  Parallel Seq Scan on opt_orders oo  (cost=0.00..12561.33 rows=155262 width=20) (actual time=0.445..129.135 rows=123479 loops=3)
                                                                  Filter: (order_date > '2024-08-18'::date)
                                                                  Rows Removed by Filter: 209855
                                                            ->  Parallel Hash  (cost=2789.35..2789.35 rows=11845 width=23) (actual time=16.397..16.399 rows=5571 loops=3)
                                                                  Buckets: 32768  Batches: 1  Memory Usage: 1216kB
                                                                  ->  Parallel Seq Scan on opt_clients oc  (cost=0.00..2789.35 rows=11845 width=23) (actual time=0.016..40.390 rows=16714 loops=1)
                                                                        Filter: (((email)::text ~~ '%@example.com'::text) AND ((status)::text = 'active'::text))
                                                                        Rows Removed by Filter: 83286
                                                      ->  Hash  (cost=350.00..350.00 rows=10000 width=10) (actual time=5.524..5.525 rows=10000 loops=3)
                                                            Buckets: 16384  Batches: 1  Memory Usage: 560kB
                                                            ->  Seq Scan on opt_products op  (cost=0.00..350.00 rows=10000 width=10) (actual time=0.873..3.575 rows=10000 loops=3)
  InitPlan 4
    ->  Limit  (cost=28580.63..28737.59 rows=1 width=32) (actual time=1227.301..1227.542 rows=1 loops=1)
          InitPlan 3
            ->  Aggregate  (cost=23906.89..23906.90 rows=1 width=8) (actual time=391.577..391.583 rows=1 loops=1)
                  ->  HashAggregate  (cost=22218.56..22968.93 rows=75037 width=21) (actual time=382.747..390.154 rows=26583 loops=1)
"                        Group Key: oc_2.name, op_2.product_name"
                        Batches: 1  Memory Usage: 4881kB
                        ->  Hash Join  (cost=8469.65..21655.78 rows=75037 width=13) (actual time=99.675..338.132 rows=61487 loops=1)
                              Hash Cond: (oo_2.product_id = op_2.product_id)
                              ->  Hash Join  (cost=7994.65..20983.72 rows=75037 width=11) (actual time=96.314..322.203 rows=61487 loops=1)
                                    Hash Cond: (oo_2.client_id = oc_2.id)
                                    ->  Bitmap Heap Scan on opt_orders oo_2  (cost=4544.31..16555.18 rows=372630 width=20) (actual time=70.916..206.135 rows=370436 loops=1)
                                          Recheck Cond: (order_date > '2024-08-18'::date)
                                          Heap Blocks: exact=7353
                                          ->  Bitmap Index Scan on idx_opt_orders_order_date  (cost=0.00..4451.15 rows=372630 width=0) (actual time=69.078..69.078 rows=370436 loops=1)
                                                Index Cond: (order_date > '2024-08-18'::date)
                                    ->  Hash  (cost=3198.63..3198.63 rows=20137 width=23) (actual time=25.203..25.205 rows=16714 loops=1)
                                          Buckets: 32768  Batches: 1  Memory Usage: 1155kB
                                          ->  Bitmap Heap Scan on opt_clients oc_2  (cost=543.43..3198.63 rows=20137 width=23) (actual time=5.653..21.114 rows=16714 loops=1)
                                                Recheck Cond: ((status)::text = 'active'::text)
                                                Filter: ((email)::text ~~ '%@example.com'::text)
                                                Rows Removed by Filter: 33245
                                                Heap Blocks: exact=1907
                                                ->  Bitmap Index Scan on idx_opt_clients_status  (cost=0.00..538.39 rows=49880 width=0) (actual time=5.329..5.329 rows=49959 loops=1)
                                                      Index Cond: ((status)::text = 'active'::text)
                              ->  Hash  (cost=350.00..350.00 rows=10000 width=10) (actual time=3.322..3.323 rows=10000 loops=1)
                                    Buckets: 16384  Batches: 1  Memory Usage: 560kB
                                    ->  Seq Scan on opt_products op_2  (cost=0.00..350.00 rows=10000 width=10) (actual time=0.026..1.612 rows=10000 loops=1)
          ->  Subquery Scan on sub2_1  (cost=4673.73..63531.88 rows=375 width=32) (actual time=1227.299..1227.532 rows=1 loops=1)
                ->  GroupAggregate  (cost=4673.73..63527.20 rows=375 width=21) (actual time=1227.288..1227.520 rows=1 loops=1)
"                      Group Key: oc_3.name, op_3.product_name"
                      Filter: (count(*) = (InitPlan 3).col1)
                      Rows Removed by Filter: 18211
                      ->  Incremental Sort  (cost=4673.73..62026.46 rows=75037 width=13) (actual time=75.948..819.557 rows=42856 loops=1)
"                            Sort Key: oc_3.name, op_3.product_name"
                            Presorted Key: oc_3.name
                            Full-sort Groups: 310  Sort Method: quicksort  Average Memory: 27kB  Peak Memory: 27kB
                            Pre-sorted Groups: 253  Sort Method: quicksort  Average Memory: 34kB  Peak Memory: 67kB
                            ->  Nested Loop  (cost=4591.52..58534.32 rows=75037 width=13) (actual time=73.803..632.645 rows=43178 loops=1)
                                  ->  Nested Loop  (cost=4591.22..56627.67 rows=75037 width=11) (actual time=73.771..597.719 rows=43178 loops=1)
                                        ->  Gather Merge  (cost=4590.80..6885.82 rows=20137 width=23) (actual time=73.719..80.200 rows=11733 loops=1)
                                              Workers Planned: 1
                                              Workers Launched: 1
                                              ->  Sort  (cost=3590.79..3620.40 rows=11845 width=23) (actual time=34.268..36.463 rows=5866 loops=2)
                                                    Sort Key: oc_3.name
                                                    Sort Method: quicksort  Memory: 1440kB
                                                    Worker 0:  Sort Method: quicksort  Memory: 25kB
                                                    ->  Parallel Seq Scan on opt_clients oc_3  (cost=0.00..2789.35 rows=11845 width=23) (actual time=0.011..14.650 rows=8357 loops=2)
                                                          Filter: (((email)::text ~~ '%@example.com'::text) AND ((status)::text = 'active'::text))
                                                          Rows Removed by Filter: 41643
                                        ->  Index Scan using idx_opt_orders_client_id on opt_orders oo_3  (cost=0.42..2.43 rows=4 width=20) (actual time=0.032..0.043 rows=4 loops=11733)
                                              Index Cond: (client_id = oc_3.id)
                                              Filter: (order_date > '2024-08-18'::date)
                                              Rows Removed by Filter: 6
                                  ->  Memoize  (cost=0.30..0.32 rows=1 width=10) (actual time=0.000..0.000 rows=1 loops=43178)
                                        Cache Key: oo_3.product_id
                                        Cache Mode: logical
                                        Hits: 43078  Misses: 100  Evictions: 0  Overflows: 0  Memory Usage: 11kB
                                        ->  Index Scan using opt_products_pkey on opt_products op_3  (cost=0.29..0.31 rows=1 width=10) (actual time=0.003..0.003 rows=1 loops=100)
                                              Index Cond: (product_id = oo_3.product_id)
  InitPlan 6
    ->  Limit  (cost=28580.64..28738.09 rows=1 width=32) (actual time=476.502..476.580 rows=1 loops=1)
          InitPlan 5
            ->  Aggregate  (cost=23906.89..23906.91 rows=1 width=32) (actual time=365.249..365.255 rows=1 loops=1)
                  ->  HashAggregate  (cost=22218.56..22968.93 rows=75037 width=21) (actual time=326.684..334.805 rows=26583 loops=1)
"                        Group Key: oc_4.name, op_4.product_name"
                        Batches: 1  Memory Usage: 4881kB
                        ->  Hash Join  (cost=8469.65..21655.78 rows=75037 width=13) (actual time=60.574..273.262 rows=61487 loops=1)
                              Hash Cond: (oo_4.product_id = op_4.product_id)
                              ->  Hash Join  (cost=7994.65..20983.72 rows=75037 width=11) (actual time=55.849..247.717 rows=61487 loops=1)
                                    Hash Cond: (oo_4.client_id = oc_4.id)
                                    ->  Bitmap Heap Scan on opt_orders oo_4  (cost=4544.31..16555.18 rows=372630 width=20) (actual time=23.266..102.236 rows=370436 loops=1)
                                          Recheck Cond: (order_date > '2024-08-18'::date)
                                          Heap Blocks: exact=7353
                                          ->  Bitmap Index Scan on idx_opt_orders_order_date  (cost=0.00..4451.15 rows=372630 width=0) (actual time=21.370..21.370 rows=370436 loops=1)
                                                Index Cond: (order_date > '2024-08-18'::date)
                                    ->  Hash  (cost=3198.63..3198.63 rows=20137 width=23) (actual time=32.384..32.387 rows=16714 loops=1)
                                          Buckets: 32768  Batches: 1  Memory Usage: 1155kB
                                          ->  Bitmap Heap Scan on opt_clients oc_4  (cost=543.43..3198.63 rows=20137 width=23) (actual time=3.394..25.850 rows=16714 loops=1)
                                                Recheck Cond: ((status)::text = 'active'::text)
                                                Filter: ((email)::text ~~ '%@example.com'::text)
                                                Rows Removed by Filter: 33245
                                                Heap Blocks: exact=1907
                                                ->  Bitmap Index Scan on idx_opt_clients_status  (cost=0.00..538.39 rows=49880 width=0) (actual time=2.735..2.735 rows=49959 loops=1)
                                                      Index Cond: ((status)::text = 'active'::text)
                              ->  Hash  (cost=350.00..350.00 rows=10000 width=10) (actual time=4.683..4.683 rows=10000 loops=1)
                                    Buckets: 16384  Batches: 1  Memory Usage: 560kB
                                    ->  Seq Scan on opt_products op_4  (cost=0.00..350.00 rows=10000 width=10) (actual time=0.033..1.882 rows=10000 loops=1)
          ->  Subquery Scan on sub6  (cost=4673.73..63719.48 rows=375 width=32) (actual time=476.501..476.572 rows=1 loops=1)
                ->  GroupAggregate  (cost=4673.73..63714.79 rows=375 width=21) (actual time=476.484..476.555 rows=1 loops=1)
"                      Group Key: oc_5.name, op_5.product_name"
                      Filter: ((count(*))::numeric = (InitPlan 5).col1)
                      Rows Removed by Filter: 4
                      ->  Incremental Sort  (cost=4673.73..62026.46 rows=75037 width=13) (actual time=111.171..111.245 rows=12 loops=1)
"                            Sort Key: oc_5.name, op_5.product_name"
                            Presorted Key: oc_5.name
                            Full-sort Groups: 1  Sort Method: quicksort  Average Memory: 27kB  Peak Memory: 27kB
                            Pre-sorted Groups: 1  Sort Method: quicksort  Average Memory: 30kB  Peak Memory: 30kB
                            ->  Nested Loop  (cost=4591.52..58534.32 rows=75037 width=13) (actual time=93.293..109.237 rows=178 loops=1)
                                  ->  Nested Loop  (cost=4591.22..56627.67 rows=75037 width=11) (actual time=93.263..108.637 rows=178 loops=1)
                                        ->  Gather Merge  (cost=4590.80..6885.82 rows=20137 width=23) (actual time=93.219..93.324 rows=55 loops=1)
                                              Workers Planned: 1
                                              Workers Launched: 1
                                              ->  Sort  (cost=3590.79..3620.40 rows=11845 width=23) (actual time=43.895..43.907 rows=28 loops=2)
                                                    Sort Key: oc_5.name
                                                    Sort Method: quicksort  Memory: 1440kB
                                                    Worker 0:  Sort Method: quicksort  Memory: 25kB
                                                    ->  Parallel Seq Scan on opt_clients oc_5  (cost=0.00..2789.35 rows=11845 width=23) (actual time=0.010..17.671 rows=8357 loops=2)
                                                          Filter: (((email)::text ~~ '%@example.com'::text) AND ((status)::text = 'active'::text))
                                                          Rows Removed by Filter: 41643
                                        ->  Index Scan using idx_opt_orders_client_id on opt_orders oo_5  (cost=0.42..2.43 rows=4 width=20) (actual time=0.268..0.277 rows=3 loops=55)
                                              Index Cond: (client_id = oc_5.id)
                                              Filter: (order_date > '2024-08-18'::date)
                                              Rows Removed by Filter: 6
                                  ->  Memoize  (cost=0.30..0.32 rows=1 width=10) (actual time=0.003..0.003 rows=1 loops=178)
                                        Cache Key: oo_5.product_id
                                        Cache Mode: logical
                                        Hits: 93  Misses: 85  Evictions: 0  Overflows: 0  Memory Usage: 10kB
                                        ->  Index Scan using opt_products_pkey on opt_products op_5  (cost=0.29..0.31 rows=1 width=10) (actual time=0.003..0.003 rows=1 loops=85)
                                              Index Cond: (product_id = oo_5.product_id)
Planning Time: 96.264 ms
Execution Time: 2454.092 ms




-------------------
У оптимізованому
Subquery Scan on stats  (cost=25032.45..30293.50 rows=1 width=96) (actual time=607.016..607.040 rows=1 loops=1)
  CTE filtered_customer_product
    ->  HashAggregate  (cost=22218.56..22968.93 rows=75037 width=21) (actual time=516.083..527.814 rows=26583 loops=1)
"          Group Key: oc.name, op.product_name"
          Batches: 1  Memory Usage: 4881kB
          ->  Hash Join  (cost=8469.65..21655.78 rows=75037 width=13) (actual time=79.020..439.534 rows=61487 loops=1)
                Hash Cond: (oo.product_id = op.product_id)
                ->  Hash Join  (cost=7994.65..20983.72 rows=75037 width=11) (actual time=74.729..409.608 rows=61487 loops=1)
                      Hash Cond: (oo.client_id = oc.id)
                      ->  Bitmap Heap Scan on opt_orders oo  (cost=4544.31..16555.18 rows=372630 width=20) (actual time=41.301..212.937 rows=370436 loops=1)
                            Recheck Cond: (order_date > '2024-08-18'::date)
                            Heap Blocks: exact=7353
                            ->  Bitmap Index Scan on idx_opt_orders_order_date  (cost=0.00..4451.15 rows=372630 width=0) (actual time=38.983..38.983 rows=370436 loops=1)
                                  Index Cond: (order_date > '2024-08-18'::date)
                      ->  Hash  (cost=3198.63..3198.63 rows=20137 width=23) (actual time=33.207..33.209 rows=16714 loops=1)
                            Buckets: 32768  Batches: 1  Memory Usage: 1155kB
                            ->  Bitmap Heap Scan on opt_clients oc  (cost=543.43..3198.63 rows=20137 width=23) (actual time=3.000..27.401 rows=16714 loops=1)
                                  Recheck Cond: ((status)::text = 'active'::text)
                                  Filter: ((email)::text ~~ '%@example.com'::text)
                                  Rows Removed by Filter: 33245
                                  Heap Blocks: exact=1907
                                  ->  Bitmap Index Scan on idx_opt_clients_status  (cost=0.00..538.39 rows=49880 width=0) (actual time=2.552..2.552 rows=49959 loops=1)
                                        Index Cond: ((status)::text = 'active'::text)
                ->  Hash  (cost=350.00..350.00 rows=10000 width=10) (actual time=4.188..4.190 rows=10000 loops=1)
                      Buckets: 16384  Batches: 1  Memory Usage: 560kB
                      ->  Seq Scan on opt_products op  (cost=0.00..350.00 rows=10000 width=10) (actual time=0.051..1.873 rows=10000 loops=1)
  ->  Aggregate  (cost=2063.52..2063.53 rows=1 width=48) (actual time=551.650..551.651 rows=1 loops=1)
        ->  CTE Scan on filtered_customer_product  (cost=0.00..1500.74 rows=75037 width=8) (actual time=516.089..542.741 rows=26583 loops=1)
  SubPlan 2
    ->  Limit  (cost=1691.14..1691.15 rows=1 width=548) (actual time=28.872..28.874 rows=1 loops=1)
          ->  Sort  (cost=1691.14..1692.08 rows=375 width=548) (actual time=28.868..28.870 rows=1 loops=1)
                Sort Key: filtered_customer_product_1.name
                Sort Method: top-N heapsort  Memory: 25kB
                ->  CTE Scan on filtered_customer_product filtered_customer_product_1  (cost=0.00..1689.27 rows=375 width=548) (actual time=0.015..15.120 rows=13647 loops=1)
                      Filter: (cnt = stats.min_cnt)
                      Rows Removed by Filter: 12936
  SubPlan 3
    ->  Limit  (cost=1691.14..1691.15 rows=1 width=548) (actual time=3.818..3.820 rows=1 loops=1)
          ->  Sort  (cost=1691.14..1692.08 rows=375 width=548) (actual time=3.817..3.818 rows=1 loops=1)
                Sort Key: filtered_customer_product_2.name
                Sort Method: quicksort  Memory: 25kB
                ->  CTE Scan on filtered_customer_product filtered_customer_product_2  (cost=0.00..1689.27 rows=375 width=548) (actual time=3.265..3.765 rows=1 loops=1)
                      Filter: (cnt = stats.max_cnt)
                      Rows Removed by Filter: 26582
  SubPlan 4
    ->  Limit  (cost=1878.74..1878.74 rows=1 width=548) (actual time=22.649..22.650 rows=1 loops=1)
          ->  Sort  (cost=1878.74..1879.68 rows=375 width=548) (actual time=22.646..22.647 rows=1 loops=1)
                Sort Key: filtered_customer_product_3.name
                Sort Method: top-N heapsort  Memory: 25kB
                ->  CTE Scan on filtered_customer_product filtered_customer_product_3  (cost=0.00..1876.86 rows=375 width=548) (actual time=0.009..15.959 rows=5561 loops=1)
                      Filter: ((cnt)::numeric = stats.avg_cnt)
                      Rows Removed by Filter: 21022
Planning Time: 0.972 ms
Execution Time: 612.352 ms
