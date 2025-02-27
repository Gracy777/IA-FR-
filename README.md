import streamlit as st
from face_analyzer import FaceAnalyzer  # Assuming this is your face analysis module

st.title("Face Analysis Tool")
st.write("Upload two images to compare faces and analyze their characteristics")

# Initialize FaceAnalyzer
analyzer = FaceAnalyzer()

# Create two columns for image upload
col1, col2 = st.columns(2)

with col1:
    st.subheader("First Image")
    image1 = st.file_uploader("Upload first image", type=['jpg', 'jpeg', 'png'], key="image1")

with col2:
    st.subheader("Second Image")
    image2 = st.file_uploader("Upload second image", type=['jpg', 'jpeg', 'png'], key="image2")

if image1 and image2:
    # Read image bytes once
    img1_data = image1.read()
    img2_data = image2.read()
    image1.seek(0)
    image2.seek(0)

    # Validate images
    valid1, msg1 = analyzer.validate_image(img1_data)
    valid2, msg2 = analyzer.validate_image(img2_data)

    if not valid1 or not valid2:
        st.error(f"Image validation failed: {msg1 if not valid1 else msg2}")
        st.stop()

    # Display uploaded images
    col1.image(image1, caption="First Image", use_column_width=True)
    col2.image(image2, caption="Second Image", use_column_width=True)

    # Show progress
    with st.spinner("Analyzing faces..."):
        result = analyzer.analyze_faces(img1_data, img2_data)

    if result.get('success', False):
        st.success("Analysis Complete!")

        # Create metric columns
        metric_col1, metric_col2, metric_col3, metric_col4 = st.columns(4)

        with metric_col1:
            st.metric("Face Similarity", f"{result['similarity_score']:.2f}%")
        with metric_col2:
            st.metric("Blur Difference", f"{result['blur_difference']:.2f}")
        with metric_col3:
            st.metric("Rotation Difference", f"{result['rotation_difference']:.2f}°")
        with metric_col4:
            st.metric("Noise Difference", f"{result['noise_difference']:.2f}")

        # Display detailed analysis
        with st.expander("Detailed Analysis"):
            st.write("Face Similarity Score:", f"{result['similarity_score']:.2f}%")
            st.write("This score indicates how similar the faces are, with 100% being an exact match.")
            st.write("\nImage Quality Analysis:")
            st.write(f"- Blur Difference: {result['blur_difference']:.2f}")
            st.write(f"- Rotation Difference: {result['rotation_difference']:.2f}°")
            st.write(f"- Noise Difference: {result['noise_difference']:.2f}")
            st.write("\nInterpretation:")
# Sidebar instructions
with st.sidebar:
    st.header("Instructions")
    st.write("""
    1. Upload two images containing faces
    2. Images should be in JPG or PNG format
    3. Maximum file size: 5MB
    4. Each image should contain at least one clear face
    5. Wait for the analysis to complete
            st.write("- Lower blur difference indicates similar image clarity")
            st.write("- Lower rotation difference indicates similar face angles")
            st.write("- Lower noise difference indicates similar image quality")
    else:
        st.error(f"Analysis failed: {result.get('error', 'Unknown error')}")

# Sidebar instructions
with st.sidebar:
    st.header("Instructions")
    st.write("""
    1. Upload two images containing faces
    2. Images should be in JPG or PNG format
    3. Maximum file size: 5MB
    4. Each image should contain at least one clear face
    5. Wait for the analysis to complete
    """)

    st.header("About")
    st.write("""
    This tool analyzes and compares faces in images using:
    - Face detection and recognition
    - Blur detection
    - Rotation analysis
    - Noise level comparison
    """)
