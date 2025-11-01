# DataUnify
Data Format Conversion
I am unable to afford the upgrade plan of replit. That is why I am uploading my project from github

import json
from datetime import datetime, timezone

# Helper to load data from the files
def load_data(file_path):
    with open(file_path, 'r') as f:
        return json.load(f)

# Helper function to convert ISO 8601 string to milliseconds timestamp
def iso_to_ms_timestamp(iso_str):
    # IMPLEMENT: Convert ISO 8601 string (e.g., "2023-10-27T10:00:00.000Z") to milliseconds timestamp (int)
    # The 'Z' indicates UTC time. datetime.fromisoformat() in Python 3.7+ can parse most ISO formats,
    # but for simplicity and consistency with standard ISO-with-Z, we'll replace 'Z' with '+00:00'.
    # If the environment is older than Python 3.7, you would use datetime.strptime()
    
    # 1. Replace 'Z' with '+00:00' for proper parsing as UTC timezone-aware object
    time_str_with_offset = iso_str.replace('Z', '+00:00')
    
    try:
        # 2. Parse the string into a datetime object
        dt_object = datetime.fromisoformat(time_str_with_offset)
        
        # 3. Ensure the object is in UTC (it should be due to '+00:00', but it's good practice)
        # Note: fromisoformat parses the timezone offset and makes it timezone-aware.
        
        # 4. Get the Unix timestamp (seconds since epoch as a float)
        # .timestamp() automatically adjusts to UTC if the datetime object is timezone-aware
        unix_seconds = dt_object.timestamp()
        
        # 5. Convert seconds to milliseconds and cast to an integer
        ms_timestamp = int(unix_seconds * 1000)
        
        return ms_timestamp
        
    except ValueError as e:
        print(f"Error converting ISO string '{iso_str}': {e}")
        return None

# IMPLEMENT: Function to unify the data formats
def unify_data(data1, data2):
    """
    Unifies data from two different formats into a single, standardized format.
    
    Args:
        data1 (dict): Data from data-1.json (milliseconds timestamp format).
        data2 (dict): Data from data-2.json (ISO 8601 timestamp format).
        
    Returns:
        dict: The unified data in the target format (data-result.json).
    """
    
    # Target format keys: 'timestamp', 'device_id', 'telemetry_data'
    
    # --- Process data1 (already in target timestamp format) ---
    unified_data1 = {
        'timestamp': data1.get('ts'), # 'ts' is already in milliseconds
        'device_id': data1.get('deviceId'),
        # Create a new dictionary for 'telemetry_data'
        'telemetry_data': {
            'temperature_c': data1.get('temp_c'),
            'battery_level': data1.get('batt_lvl'),
            'signal_strength': data1.get('sig_str')
        }
    }
    
    # --- Process data2 (needs timestamp conversion) ---
    iso_timestamp = data2.get('time') # Timestamp is in ISO 8601 format
    ms_timestamp = iso_to_ms_timestamp(iso_timestamp)
    
    unified_data2 = {
        'timestamp': ms_timestamp, # Use the converted timestamp
        'device_id': data2.get('id'),
        # Create a new dictionary for 'telemetry_data'
        'telemetry_data': {
            'temperature_c': data2.get('temperature_c'),
            'battery_level': data2.get('battery_level'),
            'signal_strength': data2.get('signal_strength')
        }
    }
    
    # The final result should be a list containing the two unified data objects
    # The structure of data-result.json suggests an array of records.
    return [unified_data1, unified_data2]

# --- Main Execution (The rest of the file is for testing your solution) ---

if __name__ == "__main__":
    # Define file paths
    data1_path = 'data-1.json'
    data2_path = 'data-2.json'
    expected_result_path = 'data-result.json'

    # Load data
    data1 = load_data(data1_path)
    data2 = load_data(data2_path)
    expected_result = load_data(expected_result_path)

    # Run the solution
    actual_result = unify_data(data1, data2)

    # Basic Test
    print("\n--- Running Tests ---")
    
    # 1. Check if the overall structure and content match
    if actual_result == expected_result:
        print("✅ Success: The unified data matches the expected result.")
        print("   Solution is correct! (This assumes the array order is consistent.)")
    else:
        # A more robust check for list of dicts (checking content regardless of order)
        # We need to sort both lists of dictionaries by a unique key (e.g., 'timestamp') 
        # to compare them correctly, as the order in the output list might not be guaranteed
        # but for this simple 2-item task, direct comparison is often intended.
        try:
            sort_key = lambda x: x['timestamp']
            sorted_actual = sorted(actual_result, key=sort_key)
            sorted_expected = sorted(expected_result, key=sort_key)
            
            if sorted_actual == sorted_expected:
                print("⚠️ Partial Success: The unified data is logically correct but the order in the list is different.")
            else:
                print("❌ Failure: The unified data does NOT match the expected result.")
                print("\nExpected:")
                print(json.dumps(expected_result, indent=4))
                print("\nActual:")
                print(json.dumps(actual_result, indent=4))

                # Extra check on timestamp conversion
                iso_time = data2.get('time')
                converted_ms = iso_to_ms_timestamp(iso_time)
                expected_ms = expected_result[1]['timestamp'] # Assuming data-2 maps to index 1
                
                print(f"\nTimestamp check for data-2: ISO='{iso_time}', Converted MS='{converted_ms}', Expected MS='{expected_ms}'")
                if converted_ms == expected_ms:
                     print("✅ Timestamp conversion is correct.")
                else:
                     print("❌ Timestamp conversion is incorrect.")

        except Exception as e:
            print(f"An error occurred during comparison: {e}")

    print("--- Tests Complete ---")
