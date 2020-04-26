##group by 默认取第一条，如果想根据自定义排序去取数据

$user = \App\Model\UserModel::select(\DB::raw('history.id as history_id,user.username,user.id,history.record_date,h2.weight - history.weight as less_weight'))
            ->where('user.is_on', 1)
            ->whereNotNull('history.weight')
            ->whereNotNull('h2.weight')
            ->leftJoin('history', function ($join) {
                $join->on('history.user_id', 'user.id')
                    ->whereRaw('history.id = (select id from history where user_id = user.id and record_time BETWEEN ' .
                        strtotime('-1 day', strtotime(date('Y-m-d 00:00:00'))) . ' and '
                        . strtotime('-1 day', strtotime(date('Y-m-d 23:59:59'))) . ' order by record_time desc limit 1)');
            })
            ->rightJoin('history as h2', function ($join) {
                $join->on('user.id', 'h2.user_id')
                    ->whereRaw('h2.id = (select id from history where user_id = user.id and record_time BETWEEN ' .
                        strtotime('-2 day', strtotime(date('Y-m-d 00:00:00'))) . ' and '
                        . strtotime('-2 day', strtotime(date('Y-m-d 23:59:59'))) . ' order by record_time desc limit 1)');
            })
            ->orderBy('less_weight', 'DESC')
            ->groupBy('user.id')
            ->get();
