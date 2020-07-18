from flask import Flask, jsonify, request, abort, flash, redirect, render_template, url_for
from werkzeug.utils import secure_filename
import os
import uuid
import csv
import time

from mysql_helper import MYSQL_helper
import PastecLib
import config
import initialize

app = Flask(__name__)

UPLOAD_FOLDER = 'media'
TEMP_FOLDER = config.TEMP_UPLOAD
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg'}
#ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['TEMP_FOLDER'] = TEMP_FOLDER

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS


def process_image(image_id, full_path):
    pastec_connection = PastecLib.PastecConnection(config.PASTEC_HOST, config.PASTEC_PORT)
    result = pastec_connection.indexImageFile(image_id, full_path)
    pastec_connection.writeIndex(config.INDEX_FILE)
    return result

def query_image(full_path):
    pastec_connection = PastecLib.PastecConnection(config.PASTEC_HOST, config.PASTEC_PORT)
    result = pastec_connection.imageQueryFile(full_path)
    return result


##  API Section start
@app.route('/upload_image', methods=['POST'])
def upload_image():
    if request.method == 'POST':
        result = None
        time1 = time.time()
        content = request.form
        print(content)
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit an empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            filename = str(uuid.uuid4())+"_"+filename
            full_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            relative_path = full_path.replace(config.PROJECT_PATH,"")
            file.save(full_path)

            sql = 'INSERT INTO `images` (file_name, file_path, process_status, active_status) VALUES(%s, %s, %s, %s)'
            val = (filename, relative_path, 1,1)
            db, cursor = MYSQL_helper().getCursor()
            cursor.execute(sql, val)
            last_id = cursor.lastrowid
            db.commit()
            result = process_image(last_id, full_path)
            
        time2 = time.time()
        diff = time2-time1
        result = {"message":result,"processing_time":diff}
        return jsonify(result), 200

@app.route('/query_image', methods=['POST','GET'])
def query_upload_image():
    if request.method == 'POST':
        time1 = time.time()
        content = request.form
        imagefile = request.files.get('img', '')
        
        if 'img' not in request.files:
            flash('No file part')
            return redirect(request.url)

        file = request.files['img']
        # if user does not select file, browser also
        # submit an empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            filename = str(uuid.uuid4())+"_"+filename
            full_path = os.path.join(app.config['TEMP_FOLDER'], filename)
            file.save(full_path)
            try:
                result = query_image(full_path)
                final_result = []
            except Exception as e:
                err = "<p>Error: %s</p>" % str(e)

                return render_template("err.html",
                                    err=err)
                
                
            
            if len(result)>0:
                for each_result in result:
                    sql = 'SELECT id, file_name, file_path, active_status FROM `images` WHERE id=%s'
                    val = (each_result[0],)
                    db, cursor = MYSQL_helper().getCursor()
                    cursor.execute(sql, val)
                    result_query = cursor.fetchone()
                    print(result_query)
                    if result_query:
                        result_dict = {"id":result_query[0], "file_name":result_query[1], "file_path":result_query[2], "active_status":result_query[3], "score":each_result[2]}
                        final_result.append(result_dict)
                        # if len(final_result) >= 1:
                        #     break
            csv_file = os.path.join(app.config['TEMP_FOLDER'], 'search-result20.csv')
            with open(csv_file, mode='w') as csv_file:
                n = 1
                # print("Open 1")
                csv_file_writer = csv.writer(csv_file, delimiter=',', quotechar='"', quoting=csv.QUOTE_MINIMAL)
                csv_file_writer.writerow(['id', 'file_name', 'score'])
                # csv_file_writer.writerow(['id', 'file_name', 'file_path', 'score'])
                for each_data in final_result:
                    print(each_data)
                    print(len(final_result))
                    # n = 1
                    # csv_writer.writerow(new_words)
                    if n < 31:
                        csv_file_writer.writerow([each_data['id'], each_data['file_name'], each_data['score']])
                        # csv_file_writer.writerow([each_data['id'], each_data['file_name'], each_data['file_path'], each_data['score']])
                    else:
                        break
                    
                    n = n+1
                    print("n = ", n)
                    

            
        time2 = time.time()
        diff = time2-time1
        print("Total time to search and write in csv: " , diff)
        result = {"message":final_result[:30],"processing_time":diff, "count":len(final_result), "count1": n - 1}
    
        return render_template("optionQuality.html",
                                    result=result)

    else:
        return "get method"
## API section stop

## WWW section start

@app.route("/")
def home():
    return render_template('homepage.html')
    
## WWW section stop

if __name__ == "__main__":
    app.run(host='127.0.0.1', port='8000', debug=True)


