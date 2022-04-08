Refactored Code

***************************************************************
Task 1 : Refactor
***************************************************************

Explanation

Code:
$cuser = User::find($user_id);

Explaination : if cuser is not available or empty

Refactored:
$cuser = User::find($user_id);
if(!isset($cuser) || empty($cuser)) {
	abort(404)
}

***************************************************************

Code:
if ($cuser && $cuser->is('customer')) { }

Explaination : double check is not compusory 

Refactored:
if ($cuser->is('customer')) { }

***************************************************************

Code:
if ($cuser && $cuser->is('customer')) {
	$jobs = $cuser->jobs()->with('user.userMeta', 'user.average', 'translatorJobRel.user.average', 'language', 'feedback')->whereIn('status', ['pending', 'assigned', 'started'])->orderBy('due', 'asc')->get();
        $usertype = 'customer';
} elseif ($cuser && $cuser->is('translator')) {
        $jobs = Job::getTranslatorJobs($cuser->id, 'new');
        $jobs = $jobs->pluck('jobs')->all();
        $usertype = 'translator';
}

Explaination : we should make a functions to identify the user type

Refactored:
$usertype = $this->getUserType($cuser);
$jobs = $this->getJobs($cuser);

public function getUserType($cuser)
{
	if ($cuser->is('customer')) 
	{							
            $usertype = 'customer';
	} elseif ($cuser->is('translator')) {
	    $usertype = 'translator';
	} else {
	    $usertype = "";
	}
		
	return $usertype;
}
	
public function getJobs($cuser)
{
	if ($cuser->is('customer')) 
	{
            $jobs = $cuser->jobs()
			->with([
				'user.userMeta', 
				'user.average', 
				'translatorJobRel.user.average', 
				'language', 
				'feedback'
			])
			->whereIn('status', ['pending', 'assigned', 'started'])
			->orderBy('due', 'asc')
			->get();
							
	} elseif($cuser->is('translator')) {
			
		$jobs = Job::getTranslatorJobs($cuser->id, 'new');
            	$jobs = $jobs->pluck('jobs')->all();
			
	} else {
			
		$jobs = NULL;
	}
		
	return $jobs;
}

***************************************************************

Code:
$job->user_email = @$data['user_email'];

Explaination : its a laravel function but if we use below code if no email exist we should handle icon or text

Refactored:
if(isset($data['user_email'])) {
	$job->user_email = $data['user_email'];
} else {
	$job->user_email = "";
}

***************************************************************

Code:
$this->mailer->send($email, $name, $subject, 'emails.job-created', $send_data);

Explaination : should use try and catch block

Refactored: 
try {
     $this->mailer->send($email, $name, $subject, 'emails.job-created', $send_data);
} catch(\Exception $e) {
     $e->getMessage();	
}

***************************************************************

Code:
$job_type = 'unpaid';
if ($translator_type == 'professional')
   $job_type = 'paid';   /*show all jobs for professionals.*/
else if ($translator_type == 'rwstranslator')
   $job_type = 'rws';  /* for rwstranslator only show rws jobs. */
else if ($translator_type == 'volunteer')
   $job_type = 'unpaid';  /* for volunteers only show unpaid jobs. */

Explaination : should use switch statement because switch statement execute fast

Refactored:
switch($translator_type) {
	case 'professional': 
		return $job_type = 'paid';
		break;
	case 'rwstranslator': 
		return $job_type = 'rws';
		break;
	case 'volunteer': 
		return $job_type = 'unpaid';
		break;
	default
		return $job_type = 'unpaid';
}

***************************************************************

Code: 
$response = curl_exec($ch);

Explaination : some time curl give empty or notgive success code so we should handle this try and catch

Refactored:
try {
    $response = curl_exec($ch);
} catch(\Exception $e) {
    $e->getMessage();
}

***************************************************************

Code:
$allJobs = Job::query();

if (isset($requestdata['feedback']) && $requestdata['feedback'] != 'false') {
	$allJobs->where('ignore_feedback', '0');
        $allJobs->whereHas('feedback', function ($q) {
               $q->where('rating', '<=', '3');
        });              
}

Refactored:
$allJobs = Job::query();
$allJobs->when($requestdata['feedback'] != 'false', function ($query) {
	$query->where('ignore_feedback', '0');
        return $query->whereHas('feedback', function ($q) {
                 $q->where('rating', '<=', '3');
        });    
});

	
***************************************************************


***************************************************************
Task 2 : Tests 
***************************************************************

Explanation

Function (willExpireAt)

Code:

$due_time = Carbon::parse($due_time);
$created_at = Carbon::parse($created_at);

$difference = $due_time->diffInHours($created_at);

if($difference <= 90)
    $time = $due_time;
elseif ($difference <= 24) {
    $time = $created_at->addMinutes(90);
} elseif ($difference > 24 && $difference <= 72) {
    $time = $created_at->addHours(16);
} else {
    $time = $due_time->subHours(48);
}
return $time->format('Y-m-d H:i:s');

Refactored:

We should used try and catch block if this function have occur some error 
try {
	$due_time = Carbon::parse($due_time);
    $created_at = Carbon::parse($created_at);
	$difference = $due_time->diffInHours($created_at);
    if($difference <= 90)
        $time = $due_time;
    elseif ($difference <= 24) {
        $time = $created_at->addMinutes(90);
    } elseif ($difference > 24 && $difference <= 72) {
        $time = $created_at->addHours(16);
    } else {
       $time = $due_time->subHours(48);
    }
    return $time->format('Y-m-d H:i:s');   
} catch (\Carbon\Exceptions\InvalidFormatException $e) {
    echo 'invalid date, enduser understands the error message';
    return false;
}

***************************************************************


Function (createOrUpdate)

Code:
public function createOrUpdateTest($id = null, $request)
{
	//some code 
}

Refactored:

Transcation because in this function so many insertions and updation if middle of the function some error occur so we should rollback automatically.

public function createOrUpdateTest($id = null, $request)
{
	DB::beginTransaction();
    try {
    	DB::commit();
    }
    catch (\Exception $e) {
        DB::rollback();
        // something went wrong
    }
}

***************************************************************

Code:

$model->user_type = $request['role'];
$model->name = $request['name'];
$model->company_id = $request['company_id'] != '' ? $request['company_id'] : 0;
$model->department_id = $request['department_id'] != '' ? $request['department_id'] : 0;
$model->email = $request['email'];
$model->dob_or_orgid = $request['dob_or_orgid'];
$model->phone = $request['phone'];
$model->mobile = $request['mobile'];
if (!$id || $id && $request['password']) $model->password = bcrypt($request['password']);
    $model->detachAllRoles();
$model->save();


Refactored:
//in this all the coding stuff is in one function we should separtely this code.In this ways the code readability is good and we should also read 

$this->_userInfo($model,$request);
private function userInfo($model,$request)
{
	$model->user_type = $request['role'];
    $model->name = $request['name'];
    $model->company_id = $request['company_id'] != '' ? $request['company_id'] : 0;
    $model->department_id = $request['department_id'] != '' ? $request['department_id'] : 0;
    $model->email = $request['email'];
    $model->dob_or_orgid = $request['dob_or_orgid'];
    $model->phone = $request['phone'];
    $model->mobile = $request['mobile'];
    if (!$id || $id && $request['password']) $model->password = bcrypt($request['password']);
        $model->detachAllRoles();
    $model->save();
}
***************************************************************

Code:

$company = Company::create(['name' => $request['name'], 'type_id' => $type->id, 'additional_info' => 'Created automatically for user ' . $model->id]);

Refactored:
//in this all the coding stuff is in one function we should separtely this code.In this ways the code readability is good and we should also read

$company = $this->_companyCreate($request,$type,$model);
private function companyCreate($request,$type,$model)
{
	return Company::create(['name' => $request['name'], 'type_id' => $type->id, 'additional_info' => 'Created automatically for user ' . $model->id]);
}
*************************************************************** 

Code:

$department = Department::create(['name' => $request['name'], 'company_id' => $company->id, 'additional_info' => 'Created automatically for user ' . $model->id]);

Refactored:

//If company insertion failed any reason 

if(!empty($company))
{
	$department = $this->_departmentCreate($request,$company,$model);
}
else
{
	DB::rollback();
}

private function departmentCreate($request,$company,$model)
{
    return Department::create(['name' => $request['name'], 'company_id' => $company->id, 'additional_info' => 'Created automatically for user ' . $model->id]);
}

*************************************************************** 

Code:
$user_meta = UserMeta::firstOrCreate(['user_id' => $model->id]);
            $old_meta = $user_meta->toArray();
            $user_meta->consumer_type = $request['consumer_type'];
            $user_meta->customer_type = $request['customer_type'];
            $user_meta->username = $request['username'];
            $user_meta->post_code = $request['post_code'];
            $user_meta->address = $request['address'];
            $user_meta->city = $request['city'];
            $user_meta->town = $request['town'];
            $user_meta->country = $request['country'];
            $user_meta->reference = (isset($request['reference']) && $request['reference'] == 'yes') ? '1' : '0';
            $user_meta->additional_info = $request['additional_info'];
            $user_meta->cost_place = isset($request['cost_place']) ? $request['cost_place'] : '';
            $user_meta->fee = isset($request['fee']) ? $request['fee'] : '';
            $user_meta->time_to_charge = isset($request['time_to_charge']) ? $request['time_to_charge'] : '';
            $user_meta->time_to_pay = isset($request['time_to_pay']) ? $request['time_to_pay'] : '';
            $user_meta->charge_ob = isset($request['charge_ob']) ? $request['charge_ob'] : '';
            $user_meta->customer_id = isset($request['customer_id']) ? $request['customer_id'] : '';
            $user_meta->charge_km = isset($request['charge_km']) ? $request['charge_km'] : '';
            $user_meta->maximum_km = isset($request['maximum_km']) ? $request['maximum_km'] : '';
            $user_meta->save();

Refactored:

in this all the coding stuff is in one function we should separtely this code.In this ways the code readability is good and we should also read 

$this->_customerUserMeta($request,$model);
private function customerUserMeta($request,$model)
{
	all insertion code
}

***************************************************************

In this ways there all code are in one function we should seprate their code in a functions. 